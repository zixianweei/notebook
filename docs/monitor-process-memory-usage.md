---
comments: true
---

# 进程内存信息

本篇记录了在Linux操作系统中，已知PID时，使用C++获取进程内存使用情况的方法。

## 内存信息类型

+ `vsize`：`virtual memory size`，虚拟内存大小，位于`/proc/<pid>/stat`中，通常指进程自己可见的内存空间，不一定与物理内存对应；
+ `rss`：`resident set size`，驻留集大小，位于`/proc/<pid>/stat`中，指进程驻留在物理内存中的大小，包含了共享库的大小；
+ `urss`：`unique resident set size`，独占驻留集大小，位于`/proc/<pid>/statm`中，指进程驻留在物理内存中的大小，不包含共享库的大小；
+ `pss`：`proportional set size`，位于`/proc/<pid>/smaps`或`/proc/<pid>/smaps_rollup`(since 4.14)中，指进程驻留在物理内存中的大小，其中共享库的大小按比例分配。

## 代码实现

```c linenums="1"
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#include <linux/version.h>

#define BUFSIZE 1024

typedef unsigned long ulong;
typedef struct {
  ulong size;
  ulong rss;
  ulong urss;
  ulong pss;
} mem_info_t;

static bool retrieve_process_memory_info(pid_t pid, mem_info_t* mem_info);

int main(int argc, char* argv[]) {
  pid_t self = getpid();
  mem_info_t mem_info = {0};
  if (!retrieve_process_memory_info(self, &mem_info)) {
    perror("ERROR: cannot retrieve process memory information: pid=[%d]\n", self);
    return 1;
  }

  printf(
      "pid = %d, size = %lu (KB), rss = %lu (KB), urss = %lu (KB), pss"
      "= %lu (KB)\n",
      self, mem_info.size, mem_info.rss, mem_info.urss, mem_info.pss);

  getchar();
  return 0;
}

bool retrieve_process_memory_info(pid_t pid, mem_info_t* mem_info) {
  FILE* fd = NULL;
  char buf[BUFSIZE];

  // /proc/<pid>/stat
  // vsize and rss
  snprintf(buf, BUFSIZE - 1, "/proc/%d/stat", pid);
  buf[BUFSIZE - 1] = '\0';
  if ((fd = fopen(buf, "r")) == NULL) {
    return false;
  }

  if (fscanf(fd,
             "%*d %*s %*c %*d %*d %*d %*d %*d %*u %*u %*u %*u %*u %*lu %*lu"
             "%*d %*d %*d %*d %*u %*u %*d %lu %lu",
             &mem_info->size, &mem_info->rss) != 2) {
    fclose(fd);
    return false;
  }

  if (fclose(fd)) {
    return false;
  }

  mem_info->size /= 1024;
  mem_info->rss = mem_info->rss * sysconf(_SC_PAGESIZE) / 1024;

  // /proc/<pid>/statm
  // urss
  snprintf(buf, BUFSIZE - 1, "/proc/%d/statm", pid);
  buf[BUFSIZE - 1] = '\0';
  if ((fd = fopen(buf, "r")) == NULL) {
    return false;
  }

  if (fscanf(fd, "%*d %*u %lu", &mem_info->urss) != 1) {
    fclose(fd);
    return false;
  }

  if (fclose(fd)) {
    return false;
  }

  mem_info->urss =
      mem_info->rss - (mem_info->urss * sysconf(_SC_PAGESIZE) / 1024);

// if kernel version is less than 4.14, use /proc/<pid>/smaps
// else, use /proc/<pid>/smaps_rollup
// pss
#if LINUX_VERSION_CODE < KERNEL_VERSION(4, 14, 0)
  printf("Using /proc/<pid>/smaps\n");
  snprintf(buf, BUFSIZE - 1, "/proc/%d/smaps", pid);
  buf[BUFSIZE - 1] = '\0';
  if ((fd = fopen(buf, "r")) == NULL) {
    return false;
  }

  char line[BUFSIZE];
  char str[BUFSIZE];
  while (fgets(line, sizeof(line), fd)) {
    ulong val = 0;
    if (sscanf(line, "%31[^:]:%lu", str, &val) == 2) {
      if (strcmp(str, "Pss") == 0) {
        mem_info->pss += val;
      }
    }
  }

  if (fclose(fd)) {
    return false;
  }
#else
  printf("Using /proc/<pid>/smaps_rollup\n");
  snprintf(buf, BUFSIZE - 1, "/proc/%d/smaps_rollup", pid);
  buf[BUFSIZE - 1] = '\0';
  if ((fd = fopen(buf, "r")) == NULL) {
    return false;
  }

  char line[BUFSIZE];
  char str[BUFSIZE];
  while (fgets(line, sizeof(line), fd)) {
    unsigned long val = 0;
    if (sscanf(line, "%31[^:]:%lu", str, &val) == 2) {
      if (strcmp(str, "Pss") == 0) {
        mem_info->pss = val;
        break;
      }
    }
  }

  if (fclose(fd)) {
    return false;
  }
#endif

  return true;
}
```

## 参考链接

+ [proc(5)-Linux manual page](https://man7.org/linux/man-pages/man5/proc.5.html)
+ [KernelNewbies: Linux_4.14](https://kernelnewbies.org/Linux_4.14)
+ [KDE/ksysguard - GitHub](https://github.com/KDE/ksysguard/blob/master/ksysguardd/Linux/ProcessList.c)
+ [Add vmPSS to Process and display it as "total memory" column](https://phabricator.kde.org/D23382)
+ [Add /proc/pid/smaps_rollup](https://patchwork.kernel.org/project/linux-fsdevel/patch/20170810001557.147285-1-dancol@google.com/#20801969)
