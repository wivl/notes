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


> https://www.zhihu.com/question/325315863

对于变量或者普通的返回值来说，常见有4种 <code>auto</code> 用法，还有一种 <code>const auto &&</code> 基本没有用，不去讨论。

1. <code>auto</code>: 产生拷贝,可以修改
2. <code>auto&</code>: 左值引用，接受左值，可以修改
3. <code>const auto&</code>: const 引用,可以接受左右值，不可修改
4. <code>auto&&</code>: 万能引用，可以接受左右值，const 引用时不能修改

```cpp
int a = 100;
const int b = 100;
auto a1 = 3; // a1为int
auto a2 = a; // a2为int
auto& a3 = a;// a3为int&,左引用
auto&& a4 = a;// a4为int&,左引用
auto&& a5 = 102;// a5为int&&,右引用
auto&& a6 = b;// a6为const int
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

C++11 使用 using 引入下面这种形式写法，并且同时支持对传统 typedef 相同的功效：

```cpp
typedef int (*process)(void *);
using NewProcess = int(*)(void *);
template<typename T>
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;

int main() {
    TrueDarkMagic<bool> you;
}
```

### 变长参数模板

C++11 加入了新的表示方法， 允许任意个数、任意类别的模板参数，同时也不需要在定义时将参数的个数固定

```cpp
template<typename... Ts> class Magic;
```

模板类 Magic 的对象，能够接受不受限制个数的 typename 作为模板的形式参数，例如下面的定义

```cpp
class Magic<int,
            std::vector<int>,
            std::map<std::string,
            std::vector<int>>> darkMagic;

class Magic<> nothing;  // 参数个数为 0 模板
```

如果不希望产生的模板参数个数为 0，可以手动的定义至少一个模板参数

```cpp
template<typename Require, typename... Args> class Magic;
```

变长参数模板也能被直接调整到到模板函数上

```cpp
template<typename... Ts>
void magic(Ts... args) {
    std::cout << sizeof...(args) << std::endl;  // 使用 sizeof 计算参数个数
}
```

我们可以传递任意个参数给 magic 函数

```cpp
magic(); // 输出0
magic(1); // 输出1
magic(1, ""); // 输出2
```

其次，对参数进行解包，到目前为止还没有一种简单的方法能够处理参数包，但有两种经典的处理手法

1. 递归模板函数

递归是非常容易想到的一种手段，也是最经典的处理方法。这种方法不断递归地向函数传递模板参数，进而达到递归遍历所有模板参数的目的

```cpp
#include <iostream>
template<typename T0>
void printf1(T0 value) {
    std::cout << value << std::endl;
}
template<typename T, typename... Ts>
void printf1(T value, Ts... args) {
    std::cout << value << std::endl;
    printf1(args...);
}
int main() {
    printf1(1, 2, "123", 1.1);
    return 0;
}
```

2. 变参模板展开

在 C++17 中增加了变参模板展开的支持，于是你可以在一个函数中完成 printf 的编写

```cpp
template<typename T0, typename... T>
void printf2(T0 t0, T... t) {
    std::cout << t0 << std::endl;
    if constexpr (sizeof...(t) > 0) printf2(t...);
}
```

3. 初始化列表展开

```cpp
template<typename T0, typename... T>
void printf2(T0 t0, T... t) {
    std::cout << t0 << std::endl;
    if constexpr (sizeof...(t) > 0) printf2(t...);
}
```

### 折叠表达式

C++ 17 中将变长参数这种特性进一步带给了表达式

```cpp
#include <iostream>
template<typename ... T>
auto sum(T ... t) {
    return (t + ...);
}
int main() {
    std::cout << sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10) << std::endl;
}
```

### 非类型模板参数推导

还有一种常见模板参数形式可以让不同字面量成为模板参数，即非类型模板参数

```cpp
template <typename T, int BufSize>
class buffer_t {
public:
    T& alloc();
    void free(T& item);
private:
    T data[BufSize];
}

buffer_t<int, 100> buf; // 100 作为模板参数
```

在这种模板参数形式下，我们可以将 100 作为模板的参数进行传递

C++17 允许 auto 关键字让编译器辅助完成具体类型的推导

```cpp
template <auto value> void foo() {
    std::cout << value << std::endl;
    return;
}

int main() {
    foo<10>();  // value 被推导为 int 类型
}
```

## 2.6 面向对象

### 委托构造

C++11 引入了委托构造的概念，这使得构造函数可以在同一个类中一个构造函数调用另一个构造函数，从而达到简化代码的目的

```cpp
#include <iostream>
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() { // 委托 Base() 构造函数
        value2 = value;
    }
};

int main() {
    Base b(2);
    std::cout << b.value1 << std::endl;
    std::cout << b.value2 << std::endl;
}
```

### 继承构造

在传统 C++ 中，构造函数如果需要继承是需要将参数一一传递的，这将导致效率低下。C++11 利用关键字 using 引入了继承构造函数的概念

```cpp
#include <iostream>
class Base {
public:
    int value1;
    int value2;
    Base() {
        value1 = 1;
    }
    Base(int value) : Base() { // 委托 Base() 构造函数
        value2 = value;
    }
};
class Subclass : public Base {
public:
    using Base::Base; // 继承构造
};
int main() {
    Subclass s(3);
    std::cout << s.value1 << std::endl;
    std::cout << s.value2 << std::endl;
}
```

### 显式虚函数重载

在传统 C++ 中，经常容易发生意外重载虚函数的事情。例如

```cpp
struct Base {
    virtual void foo();
};
struct SubClass: Base {
    void foo();
};
```

<code>SubClass::foo</code> 可能并不是程序员尝试重载虚函数，只是恰好加入了一个具有相同名字的函数。另一个可能的情形是，当基类的虚函数被删除后，子类拥有旧的函数就不再重载该虚拟函数并摇身一变成为了一个普通的类方法，这将造成灾难性的后果。

C++11 引入了 <code>override</code> 和 <code>final</code> 这两个关键字来防止上述情形的发生。

1. override

当重载虚函数时，引入 override 关键字将显式的告知编译器进行重载，编译器将检查基函数是否存在这样的虚函数，否则将无法通过编译

```cpp
struct Base {
    virtual void foo(int);
};
struct SubClass: Base {
    virtual void foo(int) override; // 合法
    virtual void foo(float) override; // 非法, 父类没有此虚函数
};
```

2. final

final 则是为了防止类被继续继承以及终止虚函数继续重载引入的

```cpp
struct Base {
    virtual void foo() final;
};
struct SubClass1 final: Base {
}; // 合法

struct SubClass2 : SubClass1 {
}; // 非法, SubClass1 已 final

struct SubClass3: Base {
    void foo(); // 非法, foo 已 final
};
```

### 显式禁用默认函数

在传统 C++ 中，如果程序员没有提供，编译器会默认为对象生成默认构造函数、 复制构造、赋值算符以及析构函数。 另外，C++ 也为所有类定义了诸如 new delete 这样的运算符。 当程序员有需要时，可以重载这部分函数。

这就引发了一些需求：无法精确控制默认函数的生成行为。 例如禁止类的拷贝时，必须将复制构造函数与赋值算符声明为 private。 尝试使用这些未定义的函数将导致编译或链接错误，则是一种非常不优雅的方式。

并且，编译器产生的默认构造函数与用户定义的构造函数无法同时存在。 若用户定义了任何构造函数，编译器将不再生成默认构造函数， 但有时候我们却希望同时拥有这两种构造函数，这就造成了尴尬。

C++11 提供了上述需求的解决方案，允许显式的声明采用或拒绝编译器自带的函数。 例如

```cpp
class Magic {
    public:
    Magic() = default; // 显式声明使用编译器生成的构造
    Magic& operator=(const Magic&) = delete; // 显式声明拒绝编译器生成构造
    Magic(int magic_number);
}
```

### 强类型枚举

在传统 C++中，枚举类型并非类型安全，枚举类型会被视作整数，则会让两种完全不同的枚举类型可以进行直接的比较（虽然编译器给出了检查，但并非所有），甚至同一个命名空间中的不同枚举类型的枚举值名字不能相同，这通常不是我们希望看到的结果。

C++11 引入了枚举类（enumeration class），并使用 enum class 的语法进行声明

```cpp
enum class new_enum : unsigned int {
    value1,
    value2,
    value3 = 100,
    value4 = 100
};
```

这样定义的枚举实现了类型安全，首先他不能够被隐式的转换为整数，同时也不能够将其与整数数字进行比较， 更不可能对不同的枚举类型的枚举值进行比较。但相同枚举值之间如果指定的值相同，那么可以进行比较

```cpp
if (new_enum::value3 == new_enum::value4) {
    // 会输出
    std::cout << "new_enum::value3 == new_enum::value4" << std::endl;
}
```

在这个语法中，枚举类型后面使用了冒号及类型关键字来指定枚举中枚举值的类型，这使得我们能够为枚举赋值（未指定时将默认使用 int）

而我们希望获得枚举值的值时，将必须显式的进行类型转换，不过我们可以通过重载 << 这个算符来进行输出，可以收藏下面这个代码段

```cpp
#include <iostream>
template<typename T>
std::ostream& operator<<(
    typename std::enable_if<std::is_enum<T>::value,
        std::ostream>::type& stream, const T& e)
{
    return stream << static_cast<typename std::underlying_type<T>::type>(e);
}
```

这时，下面的代码将能够被编译

```cpp
std::cout << new_enum::value3 << std::endl
```

# 语言运行期的变化

## 3.1 Lambda 表达式

Lambda 表达式，实际上就是提供了一个类似匿名函数的特性， 而匿名函数则是在需要一个函数，但是又不想费力去命名一个函数的情况下去使用的

### 基础

```cpp
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {
// 函数体
}
```