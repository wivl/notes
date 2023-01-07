# 《现代 C++ 教程 高速上手 C++ 11/14/17/20》

# 1 迈向现代 C++

> https://changkun.de/modern-cpp/zh-cn/01-intro/

## 1.1 被弃用的特性

* 不再允许字符串字面值常量赋值给一个 <code>char *</code>，应使用 <code>const char *</code>或者 <code>auto</code>

```c++
char *str = "hello world";  // 将出现弃用警告
```

* C++98 异常说明、 <code>unexpected_handler</code>、<code>set_unexpected()</code> 等相关特性被弃用，应该使用 <code>noexcept</code>

* <code>auto_ptr</code> 被弃用，应使用 <code>unique_ptr</code>

* <code>register</code> 关键字被弃用，不具备实际意义

* *bool* 类型的 <code>++</code> 操作被弃用

* 如果一个类有析构函数，为其生成拷贝构造函数和拷贝赋值运算符的特性被弃用了

* C 语言风格的类型转换被弃用，即（convert_type），应该使用 <code>static_cast</code>、<code>reinterpret_cast</code>、<code>const_cast</code> 来进行类型转换

* C++17 标准弃用了 <code>ccomplex</code>、<code>cstdalign</code>、<code>cstdbool</code>、<code>ctgmath</code> 等

## 与 C 的兼容性

### ifdef ifndef

```cpp
#ifndef MONGOOSE_HEADER_INCLUDED
#define    MONGOOSE_HEADER_INCLUDED
 
#ifdef __cplusplus
extern "C" {
#endif /* __cplusplus */
 
/*.................................
 * do something here
 *.................................
 */
 
#ifdef __cplusplus
}
#endif /* __cplusplus */
 
#endif /* MONGOOSE_HEADER_INCLUDED */
```

### extern static

* extern: 全局变量，可以在其他模块使用
* static: 只能在本模块中使用

```cpp
//file1.c:
    int x=1;    // 定义 x
    int f(){do something here}
//file2.c:
    extern int x;   // 声明 x
    int f();    // 声明 f
    void g(){x=f();}
```

### extern "C"
> https://www.cnblogs.com/skynet/archive/2010/07/10/1774964.html

extern "C"指令中的C，表示一种编译和链接规约

```cpp
extern "C" char* strcpy(char*,const char*);
```

### C++ 中调用 C 代码

必须要加上 extern 关键字

```cpp
// cHeader.h
#ifndef C_HEADER
#define C_HEADER
 
extern void print(int i);
 
#endif C_HEADER
```

```cpp
// cHeader.c
#include <stdio.h>
#include "cHeader.h"
void print(int i)
{
    printf("cHeader %d\n",i);
}
```

在 cpp 文件中使用以上 C 代码

```cpp
extern "C"{
#include "cHeader.h"
}
 
int main(int argc,char** argv)
{
    print(3);
    return 0;
}
```

### C 调用 C++ 代码

```cpp
// cppHeader.h
#ifndef CPP_HEADER
#define CPP_HEADER
 
extern "C" void print(int i);
 
#endif CPP_HEADER
```

```cpp
// cppHeader.cpp
#include "cppHeader.h"
 
#include <iostream>
using namespace std;
void print(int i)
{
    cout<<"cppHeader "<<i<<endl;
}
```

```cpp
// c.c
#include "cppHeader.h"
extern void print(int i);
int main(int argc,char** argv)
{
    print(3);
    return 0;
}
```

# 2 语言可用性的强化

> https://changkun.de/modern-cpp/zh-cn/02-usability/

## 2.1 常量

### nullptr

**C++11** 引入了 <code>nullptr</code> 关键字，专门用来区分**空指针**、**0**。<code>nullptr</code> 的类型为 <code>nullptr_t</code>，能够隐式的转换为任何指针或成员指针的类型，也能和他们进行相等或者不等的比较。

### constexpr

即常量表达式

```cpp
#include <iostream>
#define LEN 10

int len_foo() {
    int i = 2;
    return i;
}
constexpr int len_foo_constexpr() {
    return 5;
}

constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-1)+fibonacci(n-2);    // 可以使用递归
}

int main() {
    char arr_1[10];                      // 合法
    char arr_2[LEN];                     // 合法

    int len = 10;
    // char arr_3[len];                  // 非法

    const int len_2 = len + 1;
    constexpr int len_2_constexpr = 1 + 2 + 3;
    // char arr_4[len_2];                // 非法，因为 c++ 标准中数组长度需要是常量表达式
    char arr_4[len_2_constexpr];         // 合法

    // char arr_5[len_foo()+5];          // 非法，C++98 之前的编译器无法得知 len_foo() 在运行期实际上是返回一个常数
    char arr_6[len_foo_constexpr() + 1]; // 合法

    std::cout << fibonacci(10) << std::endl;
    // 1, 1, 2, 3, 5, 8, 13, 21, 34, 55
    std::cout << fibonacci(10) << std::endl;
    return 0;
}
```

从 C++14 开始，constexpr 函数可以在内部使用局部变量、循环和分支等简单语句，例如下面的代码在 C++11 的标准下是不能够通过编译的：

```cpp
constexpr int fibonacci(const int n) {
    if(n == 1) return 1;
    if(n == 2) return 1;
    return fibonacci(n-1) + fibonacci(n-2);
}
```

为此，我们可以写出下面这类简化的版本来使得函数从 C++11 开始即可用：

```cpp
constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-1) + fibonacci(n-2);
}
```

## 2.2 变量及其初始化

### if/switch 变量声明强化

C++17 之前

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    // 在 c++17 之前
    const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 2);
    if (itr != vec.end()) {
        *itr = 3;
    }

    // 需要重新定义一个新的变量
    const std::vector<int>::iterator itr2 = std::find(vec.begin(), vec.end(), 3);
    if (itr2 != vec.end()) {
        *itr2 = 4;
    }

    // 将输出 1, 4, 3, 4
    for (std::vector<int>::iterator element = vec.begin(); element != vec.end(); 
        ++element)
        std::cout << *element << std::endl;
}
```

在 if/switch 中创建临时变量

```cpp
// 将临时变量放到 if 语句内
if (const std::vector<int>::iterator itr = std::find(vec.begin(), vec.end(), 3);
    itr != vec.end()) {
    *itr = 4;
}
```

### 初始化列表

C++11 之前

```cpp
#include <iostream>
#include <vector>

class Foo {
public:
    int value_a;
    int value_b;
    Foo(int a, int b) : value_a(a), value_b(b) {}
};

int main() {
    // before C++11
    int arr[3] = {1, 2, 3};
    Foo foo(1, 2);
    std::vector<int> vec = {1, 2, 3, 4, 5};

    std::cout << "arr[0]: " << arr[0] << std::endl;
    std::cout << "foo:" << foo.value_a << ", " << foo.value_b << std::endl;
    for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << std::endl;
    }
    return 0;
}
```

C++11 之后

```cpp
#include <initializer_list>
#include <vector>
#include <iostream>

class MagicFoo {
public:
    std::vector<int> vec;
    MagicFoo(std::initializer_list<int> list) {
        for (std::initializer_list<int>::iterator it = list.begin();
             it != list.end(); ++it)
            vec.push_back(*it);
    }
};
int main() {
    // after C++11
    MagicFoo magicFoo = {1, 2, 3, 4, 5};

    std::cout << "magicFoo: ";
    for (std::vector<int>::iterator it = magicFoo.vec.begin(); 
        it != magicFoo.vec.end(); ++it) 
        std::cout << *it << std::endl;
}
```

这种构造函数被叫做初始化列表构造函数，具有这种构造函数的类型将在初始化时被特殊关照

初始化列表除了用在对象构造上，还能将其作为普通函数的形参，例如:

```cpp
public:
    void foo(std::initializer_list<int> list) {
        for (std::initializer_list<int>::iterator it = list.begin();
            it != list.end(); ++it) vec.push_back(*it);
    }

magicFoo.foo({6,7,8,9});
```

其次，C++11 还提供了统一的语法来初始化任意的对象，例如

```cpp
Foo foo2 {3, 4};
```

### 结构化绑定

```cpp
#include <iostream>
#include <tuple>

std::tuple<int, double, std::string> f() {
    return std::make_tuple(1, 2.3, "456");
}

int main() {
    auto [x, y, z] = f();
    std::cout << x << ", " << y << ", " << z << std::endl;
    return 0;
}
```

## 2.3 类型推导

### auto

使用 auto 进行类型推导的一个最为常见而且显著的例子就是迭代器

```cpp
// 在 C++11 之前
// 由于 cbegin() 将返回 vector<int>::const_iterator
// 所以 it 也应该是 vector<int>::const_iterator 类型
for(vector<int>::const_iterator it = vec.cbegin(); it != vec.cend(); ++it)
```

而有了 auto 之后

```cpp
for (auto it = magicFoo.vec.begin(); it != magicFoo.vec.end(); ++it) {
    std::cout << *it << ", ";
}
```

其他用法

```cpp
auto i = 5;              // i 被推导为 int
auto arr = new auto(10); // arr 被推导为 int *
```

从 C++ 20 起，auto 甚至能用于函数传参

```cpp
int add(auto x, auto y) {
    return x+y;
}

auto i = 5; // 被推导为 int
auto j = 6; // 被推导为 int
std::cout << add(i, j) << std::endl;
```

> auto 还不能推导数组类型

```cpp
auto auto_arr2[10] = {arr}; // 错误, 无法推导数组元素类型
```

```
2.6.auto.cpp:30:19: error: 'auto_arr2' declared as array of 'auto'
    auto auto_arr2[10] = {arr};
```

### decltype

decltype 关键字是为了解决 auto 关键字只能对变量进行类型推导的缺陷而出现的。它的用法和 typeof 很相似

```cpp
decltype(表达式)
```

计算某个表达式的类型

```cpp
auto x = 1;
auto y = 2;
decltype(x+y) z;
```

下面这个例子就是判断上面的变量 x y z 是否是同一个类型

```cpp
if (std::is_same<decltype(x), int>::value)
    std::cout << "type x == int" << std::endl;
if (std::is_same<decltype(x), float>::value)
    std::cout << "type x == float" << std::endl;
if (std::is_same<decltype(x), decltype(z)>::value)
    std::cout << "type z == type x" << std::endl;
```

其中，std::is_same<T, U> 用于判断 T 和 U 这两个类型是否相等。输出结果为：

```cpp
type x == int
type z == type x
```

### 尾返回类型推导

传统 C++ 并不能推导函数的返回类型

```cpp
template<typename R, typename T, typename U>
R add(T x, U y) {
    return x+y;
}
```

C++11 尾返回类型利用 auto 关键字返回类型后置

```cpp
template<typename T, typename U>
auto add2(T x, U y) -> decltype(x+y){
    return x + y;
}
```

C++14 开始是可以直接让普通函数具备返回值推导

```cpp
template<typename T, typename U>
auto add3(T x, U y){
    return x + y;
}
```

### decltype(auto)

> https://changkun.de/modern-cpp/zh-cn/02-usability/#decltype-auto

## 2.4 控制流

C++17 将 constexpr 这个关键字引入到 if 语句中，允许在代码中声明常量表达式的判断条件

```cpp
#include <iostream>

template<typename T>
auto print_type_info(const T& t) {
    if constexpr (std::is_integral<T>::value) {
        return t + 1;
    } else {
        return t + 0.001;
    }
}
int main() {
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```

在编译时，实际代码就会表现为如下：

```cpp
int print_type_info(const int& t) {
    return t + 1;
}
double print_type_info(const double& t) {
    return t + 0.001;
}
int main() {
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```

### 区间 for 迭代

C++11 引入了基于范围的迭代写法，我们拥有了能够写出像 Python 一样简洁的循环语句，我们可以进一步简化前面的例子

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};
    if (auto itr = std::find(vec.begin(), vec.end(), 3); itr != vec.end()) *itr = 4;
    for (auto element : vec)
        std::cout << element << std::endl; // read only
    for (auto &element : vec) {
        element += 1;                      // writeable
    }
    for (auto element : vec)
        std::cout << element << std::endl; // read only
}
```

## 2.5 模板

模板的哲学在于将一切能够在编译期处理的问题丢到编译期进行处理，仅在运行时处理那些最核心的动态服务，进而大幅优化运行期的性能

### 外部模板

```cpp
template class std::vector<bool>;          // 强行实例化
extern template class std::vector<double>; // 不在该当前编译文件中实例化模板
```

### 尖括号 ">"

在传统 C++ 的编译器中，>>一律被当做右移运算符来进行处理。但实际上我们很容易就写出了嵌套模板的代码：

```cpp
std::vector<std::vector<int>> matrix;
```

这在传统 C++ 编译器下是不能够被编译的，而 C++11 开始，连续的右尖括号将变得合法，并且能够顺利通过编译。甚至于像下面这种写法都能够通过编译：

```cpp
template<bool T>
class MagicType {
    bool magic = T;
};

// in main function:
std::vector<MagicType<(1>2)>> magic; // 合法, 但不建议写出这样的代码
```

### 类型别名模板

在了解类型别名模板之前，需要理解『模板』和『类型』之间的不同。仔细体会这句话：模板是用来产生类型的。在传统 C++ 中，typedef 可以为类型定义一个新的名称，但是却没有办法为模板定义一个新的名称。因为，模板不是类型。例如

```cpp
template<typename T, typename U>
class MagicType {
public:
    T dark;
    U magic;
};

// 不合法
template<typename T>
typedef MagicType<std::vector<T>, std::string> FakeDarkMagic;
```