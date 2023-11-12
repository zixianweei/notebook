---
comments: true
---

# 利用C++的rfind实现startsWith功能

## startsWith

一般而言，`startsWith`接口可以用于判断一个字符串是否以某些字符串开始。在Java中，`startsWith`函数的原型[^1]为：

```java
// @param chars: A String, representing the character(s) to check for.
// @return: A boolean value:
//          true  - if the string starts with the specified character(s).
//          false - if the string does not start with the specified 
//                  character(s).
public boolean startsWith(String chars);
```

## std::rfind

C++的string类并未直接提供`startsWith`接口，但是可以使用`rfind`实现类似的效果。`rfind`的函数原型[^2]为：

```cpp
size_t rfind(const string& str, size_t pos = npos) const noexcept;
```

!!! note "cplusplus.com"

    When **pos** is specified, the search only includes sequences of characters that begin at or before position **pos**, ignoring any possible match beginning after **pos**.[^2]

这里，`pos`参数可以省略，此时`rfind`将会得到匹配结果最后出现的位置。当使用`pos`时，`rfind`只会搜索`pos`之前和`pos`所在的位置，即`rfind`不再匹配`pos`之后的位置。

因此，为了实现`startsWith`，这里可以使用`pos = 0`。此时，`rfind`将只搜索字符串开始的位置，且只搜索一次，从而实现和`startsWith`相同的效果[^3]。

## 示例代码

```cpp linenums="1"
#include <iostream>
#include <string>

inline bool startsWith(const std::string& str, const std::string& chars) {
  return str.rfind(chars, 0) == 0;
}

inline bool startsWith(const std::string& str, const char* chars) {
  return str.rfind(chars, 0) == 0;
}

inline bool startsWith(const std::string& str, char ch) {
  return str.rfind(ch, 0) == 0;
}

int main() {
  string s = "abcdefabcdef";
  std::cout << std::boolalpha << startsWith(s, "abc") << std::endl;
  std::cout << std::boolalpha << startsWith(s, "bcd") << std::endl;
  std::cout << std::boolalpha << startsWith(s, 'a') << std::endl;
  std::cout << std::boolalpha << startsWith(s, 'b') << std::endl;
  return 0;
}
```

## 新的C++17和C++20中的变化

在C++17标准带来了关于`std::string`的视图`std::string_view`。`std::string_view`本质上是不持有字符串对象，仅保留对字符串对象只读引用的类[^4]。`std::string_view`的主要目的是避免`std::string`在使用过程中可能产生的构造或拷贝行为[^5][^6]。对应的，使用`std::string_view`实现`startsWith`的内容如下，与`std::string`几乎没有区别。

```cpp linenums="1"
inline bool startsWith(std::string_view sv, std::string_view chars) {
  return sv.rfind(chars, 0) == 0;
}

inline bool startsWith(std::string_view sv, const char* chars) {
  return sv.rfind(chars, 0) == 0;
}

inline bool startsWith(std::string_view sv, char ch) {
  return sv.rfind(ch, 0) == 0;
}
```

而C++20标准则直接为`std::string`和视图`std::string_view`补充了`starts_with()`方法，用于判断字符串或字符串视图是否以特定字符序列开始[^7]。

```cpp linenums="1"
constexpr bool starts_with( basic_string_view sv ) const noexcept;  // since C++20
constexpr bool starts_with( CharT ch ) const noexcept;  // since C++20
constexpr bool starts_with( const CharT* s ) const;  // since C++20
```

[^1]: [Java String startsWith() Method](https://www.w3schools.com/java/ref_string_startswith.asp)
[^2]: [cplusplus.com - std::string::rfind](https://cplusplus.com/reference/string/string/rfind/)
[^3]: [How do I check if a C++ std::string starts with a certain string, and convert a substring to an int?](https://stackoverflow.com/questions/1878001/how-do-i-check-if-a-c-stdstring-starts-with-a-certain-string-and-convert-a)
[^4]: [Microsoft - <string_view>](https://learn.microsoft.com/zh-cn/cpp/standard-library/string-view?view=msvc-170)
[^5]: [C++17 string_view的原理](https://zhxilin.github.io/post/tech_stack/1_programming_language/modern_cpp/cpp17/string_view/)
[^6]: [C++17 特性:使用 std::string_view 时小心踩坑](https://uint128.com/2022/02/16/C-17-%E7%89%B9%E6%80%A7-%E4%BD%BF%E7%94%A8-std-string-view-%E6%97%B6%E5%B0%8F%E5%BF%83%E8%B8%A9%E5%9D%91/)
[^7]: [cppreference.com - std::basic_string_view<CharT,Traits>::starts_with](https://en.cppreference.com/w/cpp/string/basic_string_view/starts_with)
