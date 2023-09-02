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

## rfind

C++的string类并未直接提供`startsWith`接口，但是可以使用`rfind`实现类似的效果。`rfind`的函数原型[^2]为：

```cpp
size_t rfind(const string& str, size_t pos = npos) const noexcept;
```

!!! note "cppreference.com"

    When **pos** is specified, the search only includes sequences of characters that begin at or before position **pos**, ignoring any possible match beginning after **pos**.

这里，`pos`参数可以省略，此时`rfind`将会得到匹配结果最后出现的位置。当使用`pos`时，`rfind`只会搜索`pos`之前和`pos`所在的位置，即`rfind`不再匹配`pos`之后的位置。

因此，为了实现`startsWith`，这里可以使用`pos = 0`。此时，`rfind`将只搜索字符串开始的位置，且只搜索一次，从而实现和`startsWith`相同的效果[^3]。

=== "C++"

    ```cpp
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

[^1]: [Java String startsWith() Method](https://www.w3schools.com/java/ref_string_startswith.asp)
[^2]: [cplusplus.com - std::string::rfind](https://cplusplus.com/reference/string/string/rfind/)
[^3]: [How do I check if a C++ std::string starts with a certain string, and convert a substring to an int?](https://stackoverflow.com/questions/1878001/how-do-i-check-if-a-c-stdstring-starts-with-a-certain-string-and-convert-a)
