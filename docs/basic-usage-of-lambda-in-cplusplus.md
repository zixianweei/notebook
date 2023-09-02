---
comments: true
---

# C++与lambda函数

从C++11开始，利用lambda表达式可以就地构造可调用函数对象或将其作为参数传递给其他函数。Lambda表达式的语法十分轻量，不需要像普通函数一样在头文件中进行声明，编写的代码更加易于理解和维护。

<!-- more -->

## 组成部分

<p style="text-align:left;"><img src="https://learn.microsoft.com/zh-cn/cpp/cpp/media/lambdaexpsyntax.png?view=msvc-170" style="width:33%;height:33%;" /></p>

1. 捕获列表：*capture clause* (Also known as the *lambda-introducer* in the C++ specification.)
2. 参数列表：*parameter list* (Also known as the *lambda declarator*.)
3. 可变规格：*mutable specification*
4. 异常规格：*exception specification*
5. 尾随返回类型：*tailing return type*
6. Lambda表达式执行体：*lambda body*

其中，2、3、4和5为可选项。

## 变量捕获

**捕获列表**用于捕获lambda表达式外部的变量，使得这些变量在表达式内部也可以使用。按变量捕获的方式，可以分为**值捕获**和**引用捕获**两种类型。具体的使用方式如下：

```c++
[]         // 不捕获任何变量
[&]        // 按引用的方式捕获所有外部变量 - 类比引用传递
[=]        // 按值的方式捕获所有外部变量 - 类比值传递
[&, var]   // 默认为引用捕获 var使用值捕获
[=, &var]  // 默认为值捕获 var使用引用捕获
[var, var] // 重复捕获 无实际意义 编译时warning
[&, &var]  // 重复捕获 无实际意义 编译时warning
[=, this]  // 重复捕获 无实际意义 编译时warning
[this]     // 值捕获this指针 相当于引用捕获了this所指对象
[*this]    // 值捕获this所指对象 since C++17
```

考虑性能，在进行变量捕获时需要注意以下内容：

+ `[&]`和`[=]`两种方式不推荐使用，可能对性能造成较大影响，应明确指出需要捕获的变量；
+ 值捕获的变量在lambda表达式作用域内默认是*read-only*(const)；若需要进行修改需要使用`mutable`关键字；
+ 值捕获的变量在lambda表达式作用域内外是不同的，在lambda表达式内部修改值捕获变量不会影响外部的值；
+ 在成员函数中使用lambda表达式并捕获了this指针后，需要额外关注对象之间的生命周期是否同步。

## 广义lambda捕获

C++14标准扩展了lambda表达式中的变量捕获：可以在**捕获列表**中初始化lambda表达式中的变量。这使得那些无法进行拷贝构造的变量可以通过`std::move`的方式捕获到lambda表达式中。

```c++ lambda_capture_cpp11.cpp
// 下面的代码将无法通过编译std::unique_ptr无法拷贝构造和拷贝赋值
#include <iostream>
#include <memory>

// C++11无法使用std::make_unique
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params) {
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}

int main(int argc, char* argv[]) {
    auto pva = make_unique<int>(10);
    auto f = [pva]() {
        std::cout << *pva << "\n";
    };

    f();
    return 0;
}

// error: call to implicitly-deleted copy constructor of 'std::unique_ptr<int>'
```

```c++ lambda_capture_cpp14.cpp
// C++14中可以通过编译并正常运行
#include <iostream>
#include <memory>

template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params) {
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}

int main(int argc, char* argv[]) {
    auto pva = make_unique<int>(10);
    auto f = [pva = std::move(pva)]() {
        std::cout << *pva << "\n";
    };

    f();
    return 0;
}
```

## 泛型lambda

C++14开始支持lambda表达式的输入参数为`auto`类型。

```c++ generic_lambda.cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main(int argc, char* argv[]) {
    auto less = [](const auto& v1, const auto& v2) {
        return v1 < v2;
    };

    std::vector<int> va1{ 1, 4, 3, 5, 2 };
    std::sort(va1.begin(), va1.end(), less);

    std::vector<double> va2{ 1.2, 1., 1.4, 1.1, 1.3 };
    std::sort(va2.begin(), va2.end(), less);

    return 0;
}
```

## 立即调用lambda

Lambda表达式在定义的时候就可以被立即调用。

```c++ immediate_invoke_lambda.cpp
#include <iostream>

int main(int argc, char* argv[]) {
    int va = 10;
    [va]() {
        std::cout << va << "\n";
    }();
    
    return 0;
}
```

## Lambda表达式的原理

```c++ lambda_principle.cpp
#include <iostream>

int main(int argc, char* argv[]) {
    int va = 10;
    auto f = [va]() {
        std::cout << va << "\n";
    };
    
    return 0;
}
```

```c++ lambda_principle_by_cppinsight.cpp
#include <iostream>

int main(int argc, char* argv[]) {
    int va = 10;

    class __lambda_6_14 {
    public:
        inline void operator()() const {
            std::operator<<(std::cout.operator<<(va), "\n");
        }

    private:
        int va;
    public:
        // inline /*constexpr */ __lambda_6_14(__lambda_6_14 &&) noexcept = default;
        __lambda_6_14(int& _va)
            : va{ _va }
        {}
    };

    __lambda_6_14 f = __lambda_6_14(__lambda_6_14{ va });
    f.operator()();
    return 0;
}
```

## 闭包的理解

*Scott Meyers*将**lambda表达式**和**闭包**之间的关系比作**类**与**对象**之间的关系：

> The distinction between a lambda and the corresponding closure is precisely equivalent to the distinction between a class and an instance of the class. A class exists only in source code; it doesn’t exist at runtime. What exists at runtime are objects of the class type. Closures are to lambdas as objects are to classes. This should not be a surprise, because each lambda expression causes a unique class to be generated (during compilation) and also causes an object of that class type–a closure–to be created (at runtime).

**闭包**， *Closure*，是在支持*first class citizen*的编程语言中实现的一种词法绑定技术。所谓*first class citizen*，是指函数能够作为其他函数的参数、返回值、赋值给变量或存储在数据结构中。

实现上，闭包是一个结构体，包含一个函数和一个关联的环境。其中，函数是闭包的入口；关联的环境则包含**内层逻辑流**和从外部捕获得到的**自由变量**。区别闭包和函数的关键是：闭包的自由变量在捕获时即被确定，那么闭包就能够使用自由变量和内部逻辑流完成运行，而不依赖捕获时的上下文。

简单来说，闭包就是指那些可以自由访问外层作用域自由变量的函数。

C++11之前，闭包是不被语言本身支持的，但是可以使用**仿函数**来模拟，不过写起来比较麻烦。C++11支持的匿名函数则完全符合闭包的定义，扩展了C++语言的边界。此外，C++11提供的`std::bind`语法糖也能够使用占位符和普通函数构造出闭包。

> 注：关于闭包的定义其实比较晦涩，可以看文末的参考文献理解下。

## 参考链接

+ [Lambda expressions (since C++11)](https://en.cppreference.com/w/cpp/language/lambda)
+ [c++中lambda表达式用法](https://juejin.cn/post/6971740770617425933)
+ [C++ 中的 Lambda 表达式](https://learn.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=msvc-170)
+ [Closure (computer programming)](https://en.wikipedia.org/wiki/Closure_(computer_programming))
+ [C++11中的匿名函数(lambda函数,lambda表达式)](https://www.cnblogs.com/pzhfei/archive/2013/01/14/lambda_expression.html)
+ [【C++】使用可变lambda, mutable关键字](https://blog.csdn.net/Trouble_provider/article/details/90521215)
+ [C++14#Generic_lambdas](https://en.wikipedia.org/wiki/C++14#Generic_lambdas)
+ [C++ 闭包和匿名函数](https://zhuanlan.zhihu.com/p/303391384)
+ [C++的闭包(closure)](https://zhuanlan.zhihu.com/p/121628510)
+ [First-class function](https://en.wikipedia.org/wiki/First-class_function)
+ [JavaScript闭包的那些事~](https://www.cnblogs.com/MomentYY/p/15877394.html)
+ [closure（闭包）、仿函数、std::function、bind、lambda](https://blog.csdn.net/daaikuaichuan/article/details/78229315)
+ [C++ 17 constexpr 與 Lambda 表達式](https://zh-blog.logan.tw/2020/03/08/cxx-17-constexpr-and-lambda-expression/)
+ [constexpr lambda expressions in C++](https://learn.microsoft.com/en-us/cpp/cpp/lambda-expressions-constexpr?view=msvc-170)
+ [C++ lambda表达式与函数对象](https://www.jianshu.com/p/d686ad9de817)
+ [语言运行期的强化#泛型-Lambda](https://changkun.de/modern-cpp/zh-cn/03-runtime/#泛型-Lambda)
+ [C++ lambda表达式教程](https://lesleylai.info/zh/c++-lambda/)
