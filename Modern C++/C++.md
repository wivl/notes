# C++

## 信号处理

|信号|描述|
|----|----|
|SIGABRT|程序的异常终止，如调用 abort|
|SIGFPE|错误的算术运算，比如除以零或导致溢出的操作|
|SIGILL|检测非法指令|
|SIGINT|程序终止(interrupt)信号|
|SIGSEGV|非法访问内存|
|SIGTERM|发送到程序的终止请求|

```cpp
#include <iostream>
#include <csignal>
#include <unistd.h>
 
using namespace std;
 
void signalHandler( int signum ) {
    cout << "Interrupt signal (" << signum << ") received.\n";
    // 清理并关闭
    // 终止程序 
   exit(signum);  
}
 
int main () {
    int i = 0;
    // 注册信号 SIGINT 和信号处理程序
    signal(SIGINT, signalHandler);  
    while(++i){
       cout << "Going to sleep...." << endl;
       if( i == 3 ){
          raise( SIGINT);
       }
       sleep(1);
    }
    return 0;
}
```

## 多线程

### 创建线程

```cpp
#include <iostream>
#include <thread>
using namespace std;

// 一个简单的函数，作为线程的入口函数
void foo(int Z) {
    for (int i = 0; i < Z; i++) {
        cout << "线程使用函数指针作为可调用参数\n";
    }
}

// 可调用对象的类定义
class ThreadObj {
public:
    void operator()(int x) const {
        for (int i = 0; i < x; i++) {
            cout << "线程使用函数对象作为可调用参数\n";
        }
    }
};

int main() {
    cout << "线程 1 、2 、3 独立运行" << endl;

    // 使用函数指针创建线程
    thread th1(foo, 3);

    // 使用函数对象创建线程
    thread th2(ThreadObj(), 3);

    // 使用 Lambda 表达式创建线程
    thread th3([](int x) {
        for (int i = 0; i < x; i++) {
            cout << "线程使用 lambda 表达式作为可调用参数\n";
        }
    }, 3);

    // 等待所有线程完成
    th1.join(); // 等待线程 th1 完成
    th2.join(); // 等待线程 th2 完成
    th3.join(); // 等待线程 th3 完成

    return 0;
}
```

### 线程同步与互斥

#### 互斥量

```cpp
#include <mutex>

std::mutex mtx; // 全局互斥量

void safeFunction() {
    mtx.lock(); // 请求锁定互斥量
    // 访问或修改共享资源
    mtx.unlock(); // 释放互斥量
}

int main() {
    std::thread t1(safeFunction);
    std::thread t2(safeFunction);
    t1.join();
    t2.join();
    return 0;
}
```

#### 锁

```cpp
#include <mutex>

std::mutex mtx;

void safeFunctionWithLockGuard() {
    std::lock_guard<std::mutex> lk(mtx);
    // 访问或修改共享资源
}

void safeFunctionWithUniqueLock() {
    std::unique_lock<std::mutex> ul(mtx);
    // 访问或修改共享资源
    // ul.unlock(); // 可选：手动解锁
    // ...
}
```

#### 条件变量

```cpp
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void workerThread() {
    std::unique_lock<std::mutex> lk(mtx);
    cv.wait(lk, []{ return ready; }); // 等待条件
    // 当条件满足时执行工作
}

void mainThread() {
    {
        std::lock_guard<std::mutex> lk(mtx);
        // 准备数据
        ready = true;
    } // 离开作用域时解锁
    cv.notify_one(); // 通知一个等待的线程
}
```

#### 原子操作

```cpp
#include <atomic>
#include <thread>

std::atomic<int> count(0);

void increment() {
    count.fetch_add(1, std::memory_order_relaxed);
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    return count; // 应返回2
}
```


#### 线程局部存储

```cpp
#include <iostream>
#include <thread>

thread_local int threadData = 0;

void threadFunction() {
    threadData = 42; // 每个线程都有自己的threadData副本
    std::cout << "Thread data: " << threadData << std::endl;
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);
    t1.join();
    t2.join();
    return 0;
}
```

### 死锁和避免策略

死锁发生在多个线程互相等待对方释放资源，但没有一个线程能够继续执行。避免死锁的策略包括：

- 总是以相同的顺序请求资源
- 使用超时来尝试获取资源
- 使用死锁检测算法

### 线程间通信

```cpp
std::promise<int> p;
std::future<int> f = p.get_future();

std::thread t([&p] {
    p.set_value(10); // 设置值，触发 future
});

int result = f.get(); // 获取值
```


## 容器

分为三类

- 顺序容器：存储元素的序列，允许双向遍历
    - std::vector: 动态数组，支持快速随机访问
    - std::deque: 双端队列，支持快速插入和删除
    - std::list: 链表，支持快速插入和删除，但不支持随机访问

- 关联容器：存储键值对，每个元素都有一个键和一个值，并且通过键来组织元素
    - std::set: 集合，不允许重复元素
    - std::multiset: 多重集合，允许多个元素基友相同的键
    - std::map: 映射，每个键映射到一个值
    - std::multimap: 多重映射，允许多个键映射到相同的值

- 无序容器：哈希表，支持快速的查找、插入和删除
    - std::unordered_set: 无序集合
    - std::unordered_multiset: 无序多重集合
    - std::unordered_map: 无序映射
    - std::unordered_multimap: 无序多重映射


### array

大小在编译时确定，并且不允许动态改变

```cpp
#include <iostream>
#include <array>

int main() {
    // 创建一个包含 5 个整数的 std::array
    std::array<int, 5> myArray = {1, 2, 3, 4, 5};

    // 使用范围 for 循环遍历数组
    for (const auto& value : myArray) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    // 使用索引访问数组元素
    std::cout << "Element at index 2: " << myArray.at(2) << std::endl;

    // 获取数组的大小
    std::cout << "Array size: " << myArray.size() << std::endl;

    // 修改数组元素
    myArray[3] = 10;

    // 再次遍历数组以显示修改后的元素
    for (const auto& value : myArray) {
        std::cout << value << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### vector

```cpp
#include <iostream>
#include <vector>

int main() {
    // 声明一个存储整数的 vector
    std::vector<int> numbers;

    // 添加元素
    numbers.push_back(10);
    numbers.push_back(20);
    numbers.push_back(30);

    // 输出 vector 中的元素
    std::cout << "Vector contains: ";
    for (int i = 0; i < numbers.size(); ++i) {
        std::cout << numbers[i] << " ";
    }
    std::cout << std::endl;

    // 添加更多元素
    numbers.push_back(40);
    numbers.push_back(50);

    // 再次输出 vector 中的元素
    std::cout << "After adding more elements, vector contains: ";
    for (int i = 0; i < numbers.size(); ++i) {
        std::cout << numbers[i] << " ";
    }
    std::cout << std::endl;

    // 访问特定元素
    std::cout << "The first element is: " << numbers[0] << std::endl;

    // 清空 vector
    numbers.clear();

    // 检查 vector 是否为空
    if (numbers.empty()) {
        std::cout << "The vector is now empty." << std::endl;
    }

    return 0;
}
```

### list

允许在容器的任意位置快速插入和删除元素

```cpp
#include <iostream>
#include <list>

int main() {
    // 创建一个整数类型的列表
    std::list<int> numbers;

    // 向列表中添加元素
    numbers.push_back(10);
    numbers.push_back(20);
    numbers.push_back(30);

    // 访问并打印列表的第一个元素
    std::cout << "First element: " << numbers.front() << std::endl;

    // 访问并打印列表的最后一个元素
    std::cout << "Last element: " << numbers.back() << std::endl;

    // 遍历列表并打印所有元素
    std::cout << "List elements: ";
    for (std::list<int>::iterator it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 删除列表中的最后一个元素
    numbers.pop_back();

    // 再次遍历列表并打印所有元素
    std::cout << "List elements after removing the last element: ";
    for (std::list<int>::iterator it = numbers.begin(); it != numbers.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### forward_list

单向链表，只允许从一端遍历，不支持 [] 和 at() 访问

```cpp
#include <iostream>
#include <forward_list>

int main() {
    // 创建一个空的 forward_list
    std::forward_list<int> fl;

    // 在列表前端添加元素
    fl.push_front(10);
    fl.push_front(20);
    fl.push_front(30);

    // 遍历 forward_list 并输出元素
    for (auto it = fl.begin(); it != fl.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 输出结果：30 20 10

    return 0;
}
```


### deque

双端队列，索引+数据块实现

```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<int> myDeque;

    // 插入元素
    myDeque.push_back(10);
    myDeque.push_back(20);
    myDeque.push_front(5);

    // 访问元素
    std::cout << "Deque contains: ";
    for (int i = 0; i < myDeque.size(); ++i) {
        std::cout << myDeque[i] << " ";
    }
    std::cout << std::endl;

    // 删除元素
    myDeque.pop_back();
    myDeque.pop_front();

    // 再次访问元素
    std::cout << "Deque after popping: ";
    for (int i = 0; i < myDeque.size(); ++i) {
        std::cout << myDeque[i] << " ";
    }
    std::cout << std::endl;

    return 0;
}
```


### stack

底层容器为 vector 或 deque

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> s;

    // 向栈中添加元素
    s.push(1);
    s.push(2);
    s.push(3);

    // 访问栈顶元素
    std::cout << "Top element is: " << s.top() << std::endl;

    // 移除栈顶元素
    s.pop();
    std::cout << "After popping, top element is: " << s.top() << std::endl;

    // 检查栈是否为空
    if (!s.empty()) {
        std::cout << "Stack is not empty." << std::endl;
    }

    // 打印栈的大小
    std::cout << "Size of stack: " << s.size() << std::endl;

    return 0;
}
```

### queue

底层为链表或动态数组

```cpp
#include <iostream>
#include <queue>

int main() {
    // 创建一个整数队列
    std::queue<int> q;

    // 向队列中添加元素
    q.push(10);
    q.push(20);
    q.push(30);

    // 打印队列中的元素数量
    std::cout << "队列中的元素数量: " << q.size() << std::endl;

    // 打印队首元素
    std::cout << "队首元素: " << q.front() << std::endl;

    // 打印队尾元素
    std::cout << "队尾元素: " << q.back() << std::endl;

    // 移除队首元素
    q.pop();
    std::cout << "移除队首元素后，队首元素: " << q.front() << std::endl;

    // 再次打印队列中的元素数量
    std::cout << "队列中的元素数量: " << q.size() << std::endl;

    return 0;
}
```


### priority_queue

优先队列，默认大根堆

```cpp
#include <iostream>
#include <queue>
#include <vector>

struct compare {
    bool operator()(int a, int b) {
        return a > b; // 定义最小堆
    }
};

int main() {
    // 创建一个自定义类型的优先队列，使用最小堆
    std::priority_queue<int, std::vector<int>, compare> pq_min;

    // 向优先队列中添加元素
    pq_min.push(30);
    pq_min.push(10);
    pq_min.push(50);
    pq_min.push(20);

    // 输出队列中的元素
    std::cout << "最小堆中的元素：" << std::endl;
    while (!pq_min.empty()) {
        std::cout << pq_min.top() << std::endl;
        pq_min.pop();
    }

    return 0;
}
```

### set

```cpp
#include <iostream>
#include <set>

int main() {
    // 声明一个整型 set 容器
    std::set<int> mySet;

    // 插入元素
    mySet.insert(10);
    mySet.insert(20);
    mySet.insert(30);
    mySet.insert(40);

    // 输出 set 中的元素
    std::cout << "Set contains: ";
    for (int num : mySet) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 查找元素
    if (mySet.find(20) != mySet.end()) {
        std::cout << "20 is in the set." << std::endl;
    } else {
        std::cout << "20 is not in the set." << std::endl;
    }

    // 删除元素
    mySet.erase(20);

    // 再次输出 set 中的元素
    std::cout << "After erasing 20, set contains: ";
    for (int num : mySet) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 检查 set 是否为空
    if (mySet.empty()) {
        std::cout << "The set is empty." << std::endl;
    } else {
        std::cout << "The set is not empty." << std::endl;
    }

    // 输出 set 中元素的数量
    std::cout << "The set contains " << mySet.size() << " elements." << std::endl;

    return 0;

}
```

### unordered_set

基于哈希表的容器

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    // 创建一个整数类型的 unordered_set
    std::unordered_set<int> uset;

    // 插入元素
    uset.insert(10);
    uset.insert(20);
    uset.insert(30);

    // 打印 unordered_set 中的元素
    std::cout << "Elements in uset: ";
    for (int elem : uset) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;

    // 查找元素
    auto it = uset.find(20);
    if (it != uset.end()) {
        std::cout << "Element 20 found in uset." << std::endl;
    } else {
        std::cout << "Element 20 not found in uset." << std::endl;
    }

    // 删除元素
    uset.erase(20);
    std::cout << "After erasing 20, elements in uset: ";
    for (int elem : uset) {
        std::cout << elem << " ";
    }
    std::cout << std::endl;

    // 检查大小和是否为空
    std::cout << "Size of uset: " << uset.size() << std::endl;
    std::cout << "Is uset empty? " << (uset.empty() ? "Yes" : "No") << std::endl;

    // 清空 unordered_set
    uset.clear();
    std::cout << "After clearing, is uset empty? " << (uset.empty() ? "Yes" : "No") << std::endl;

    return 0;
}
```


### map

```cpp
#include <iostream>#include <map>
#include <string>

int main() {
    // 创建一个 map 容器，存储员工的姓名和年龄
    std::map<std::string, int> employees;

    // 插入员工信息
    employees["Alice"] = 30;
    employees["Bob"] = 25;
    employees["Charlie"] = 35;

    // 遍历 map 并打印员工信息
    for (std::map<std::string, int>::iterator it = employees.begin(); it != employees.end(); ++it) {
        std::cout << it->first << " is " << it->second << " years old." << std::endl;
    }

    return 0;
}
```

### unordered_map

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    // 创建一个 unordered_map，键为 int，值为 string
    std::unordered_map<int, std::string> myMap;

    // 插入一些键值对
    myMap[1] = "one";
    myMap[2] = "two";
    myMap[3] = "three";

    // 打印所有元素
    for (const auto& pair : myMap) {
        std::cout << "Key: " << pair.first << ", Value: " << pair.second << std::endl;
    }

    // 访问特定键的值
    std::cout << "Value for key 2: " << myMap[2] << std::endl;

    // 删除键为1的元素
    myMap.erase(1);

    // 再次打印所有元素
    std::cout << "After erasing key 1:" << std::endl;
    for (const auto& pair : myMap) {
        std::cout << "Key: " << pair.first << ", Value: " << pair.second << std::endl;
    }

    return 0;
}
```


### bitset

位集合是一个由位（bit）组成的数组，每个位可以是0或1。bitset 类型非常适合于需要表示二进制数据或进行位操作的场景

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b("11001010"); // 从字符串初始化
    std::cout << "Initial bitset: " << b << std::endl;

    // 访问特定位置的位
    std::cout << "Bit at position 3: " << b[3] << std::endl;

    // 修改位
    b[3] = 1;
    std::cout << "Modified bitset: " << b << std::endl;

    // 翻转位
    b.flip();
    std::cout << "Flipped bitset: " << b << std::endl;

    return 0;
}
```

```cpp
#include <iostream>
#include <bitset>

int main() {
    std::bitset<8> b1("10101010");
    std::bitset<8> b2("11110000");

    // 位与操作
    std::bitset<8> b_and = b1 & b2;
    std::cout << "Bitwise AND: " << b_and << std::endl;

    // 位或操作
    std::bitset<8> b_or = b1 | b2;
    std::cout << "Bitwise OR: " << b_or << std::endl;

    // 位异或操作
    std::bitset<8> b_xor = b1 ^ b2;
    std::cout << "Bitwise XOR: " << b_xor << std::endl;

    // 位非操作
    std::bitset<8> b_not = ~b1;
    std::cout << "Bitwise NOT: " << b_not << std::endl;

    return 0;
}
```

## 算法库 algorithm

### sort

### find

### copy

### equal


## functional

### function

函数封装器

```cpp
#include <iostream>
#include <functional>

void greet() {
    std::cout << "Hello, World!" << std::endl;
}

int main() {
    std::function<void()> f = greet; // 使用函数
    f(); // 输出: Hello, World!

    std::function<void()> lambda = []() {
        std::cout << "Hello, Lambda!" << std::endl;
    };
    lambda(); // 输出: Hello, Lambda!

    return 0;
}
```

### bind

std::bind 允许我们创建一个可调用对象，它在调用时会将给定的参数绑定到一个函数或函数对象

```cpp
#include <iostream>
#include <functional>

int add(int a, int b) {
    return a + b;
}

int main() {
    auto bound_add = std::bind(add, 5, std::placeholders::_1); // 占位符
    std::cout << bound_add(10) << std::endl; // 输出: 15

    return 0;
}
```
