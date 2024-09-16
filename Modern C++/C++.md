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
