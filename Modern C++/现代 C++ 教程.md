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
