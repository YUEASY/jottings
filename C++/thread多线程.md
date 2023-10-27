# C++ thread

- [Atomic ](https://legacy.cplusplus.com/reference/atomic/)

- [Thread ](https://legacy.cplusplus.com/reference/thread/)

- [Mutex ](https://legacy.cplusplus.com/reference/mutex/)

- [Condition variable](https://legacy.cplusplus.com/reference/condition_variable/)

- [Future](https://legacy.cplusplus.com/reference/future/) (暂缺)

----

## \<thread\>

### thread （主线程操作）

#### 创建线程

```C++
// 无参构造（用于线程池）
std::thread t();
// 绑定函数（利用万能引用）
std::thread t1(函数对象[函数指针、仿函数...],参数1,参数2，...);
// 资源转移
std::thread t2(t1);
// 拷贝构造已禁用
```

>  注：由于可变参数的原因，传引用需要使用 std::ref( )

#### 获取线程id

```c++
std::thread t1(...);
t1.git_id();
```

#### 回收线程

```c++
// 阻塞式回收
// join  
// 若子进程处于运行状态则主进程一直等待
std::thread t1(...);
t1.join;

// 非阻塞回收
// joinable 
// 判断子进程是否进入僵尸状态
std::thread t1(...);
if(t1.joinable())t1.join;
```

### this_thread （子线程操作）

#### 获取线程id

```c++
// 在子进程中调用
std::this_thread::git_id();
```

#### 减少时间片的占用

```c++
std::this_thread::yield();
```

#### 休眠等待

```c++
#include <chrono> 

// sleep_until
std::this_thread::sleep_until(时间戳);

// 获取当前时间戳
// using namespace std::chrono;
// system_clock::time_point now = system_clock::now();

// sleep_for
std::this_thread::sleep_for(时间段);
// 时间段
// std::chrono::seconds(1)			1s
// std::chrono::milliseconds(10)	10ms
```



## \<mutex\> 

### mutex （互斥锁）

```c++
// 创建锁
std::mutex mtx;
// 加锁
mtx.lock();
// 尝试是否能申请到锁
if(mtx.try_lock())
    mtx.lock;
// 解锁
mtx.unlock;
```

> 注：mutex 禁用拷贝构造函数，故传递时需要 引用 或 指针

### recursive_mutex （用于递归的互斥锁）

```c++
// 只关注不同线程，对于同一线程再次加锁时不会等待而会忽略
std::recursive_mutex res_mtx;
```

### lock_guard （轻量级互斥锁封装）

```c++
std::mutex mtx;
{
    // 创建后自动加锁mtx
	std::lock_guard<std::mutex> lock(mtx);
}
// lock_guard析构时自动解锁mtx
```

```c++
// 简单模拟实现 利用RAII思想
template<class Lock>
class LockGuard
{
public:
    LockGuard(Lock& lock):_lock(lock){_lock.look();}
    ~LockGuard(){_lock.unlock();}
private:
    Lock& _lock;
};
std::mutex mtx;
LockGuard<std::mutex> lock(mtx);
```

### unique_lock （互斥锁封装）

#### 创建锁

```c++
std::mutex mtx;

{
    std::unique_lock<std::mutex> lock (mtx[,可选参数]);
	// 无: 阻塞式加锁 (lock)
	// try_to_lock: 尝试加锁 (try_lock)
	// defer_lock: 不加锁
	// adopt_lock: 已加锁
}
// 析构时自动解锁
```

#### 加锁和解锁

```c++
// 加锁
lock.lock();
lock.try_lock();
lock.try_lock_for();
lock.try_lock_until();

// 解锁
lock.unlock();
```



## \<atmoic\>	

### 原子量

```c++
// 创建原子量
std::atomic<int> a(10);
std::atomic<bool> flag = true;
// 只支持基本数据类型
```

使用 `CAS` 原理执行**原子操作**。

> CAS ：[Compare And Swap](https://en.wikipedia.org/wiki/Compare-and-swap)
>
> 原子操作：指在多线程环境中可以安全地执行的操作，不会被其他线程的操作干扰，不被中断。



## <condition_variable\>

### 条件变量

#### 创建条件变量

```c++
std::condition_variable cv;
```

#### 线程等待

```c++
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx);
// 传入锁
// 进入等待直到唤醒
cv.wait(lock);
// 根据函数返回值来判断是否进入等待(true/false)
// false:进入等待;true:直接略过
cv.wait(lock,条件函数)；
// 时间相关
cv.wait_for(lock,时间段);
cv.wait_until(lock,时间戳);
```

> 注：1、条件变量不是线程安全的，故需要传入互斥锁
>
> ​		2、wait 阻塞会对 lock 进行解锁 unlock()，来让其他线程继续
>
> ​		3、wait 阻塞取消则会对 lock 进行加锁 lock()

#### 线程唤醒

```c++
// 唤醒一个线程,如果有多个线程在等待，则随机唤醒
cv.notify_one();
// 唤醒所有线程
cv.notify_all();
```



## 使用示例

### 交替打印奇偶数

#### 1. 借助 yield()

```c++
#include <thread>
#include <iostream>
using namespace std;
int main()
{
    int i = 0;
    int n = 100;
    thread t1([&]() {
        while (i<n)
        {	
            //打印偶数
            while (i % 2 != 0)this_thread::yield();
            cout << this_thread::get_id() << ":" << i << endl;
            i++;
        }});

    thread t2([&]() {
        while (i < n)
        {
            //打印奇数
            while (i % 2 == 0)this_thread::yield();
            cout << this_thread::get_id() << ":" << i << endl;
            i++;
        }});

    t1.join();
    t2.join();
    return 0;
}
```

#### 2. mutex互斥锁

```c++
#include <thread>
#include <mutex>
#include <iostream>
using namespace std;

int main()
{
	mutex mtx;

	int i = 0;
	int n = 100;
	thread t1([&]() {
		//cout << "t1:" << this_thread::get_id() << endl;
		while (i < n)
		{
			//打印偶数
			if (i % 2 == 0)
			{
				mtx.lock();
				cout << this_thread::get_id() << ":" << i << endl;
				i++;
				mtx.unlock();
			}
		}});

	thread t2([&]() {
		//cout << "t2:" << this_thread::get_id() << endl;
		while (i < n)
		{
			//打印奇数
			if (i % 2 != 0)
			{
				mtx.lock();
				cout << this_thread::get_id() << ":" << i << endl;
				i++;
				mtx.unlock();
			}
		}});

	t1.join();
	t2.join();
	return 0;
}
```

#### 3. 使用条件变量

```c++
#include <iostream>
#include <condition_variable>
#include <thread>
#include <mutex>
using namespace std;

int main()
{
	mutex mtx;
	condition_variable cv;

	int i = 0;
	int n = 10;
	thread t1([&]() {
		while (i < n)
		{
			//打印奇数	
			unique_lock<mutex> lock(mtx);//自动加锁，生命周期解锁自动析构
			cv.wait(lock, [&i]() {return i%2;});
			cout << this_thread::get_id() << ":" << i << endl;
			i++;
			cv.notify_one();
		}});

	thread t2([&]() {
		while (i < n)
		{
			//打印偶数
			unique_lock<mutex> lock(mtx);
			cv.wait(lock, [&i]() {return !(i%2); });
			cout << this_thread::get_id() << ":" << i << endl;
			i++;
			cv.notify_one();
		}});


	cout << "t1:" << t1.get_id() << endl;
	cout << "t2:" << t2.get_id() << endl;
	t1.join();
	t2.join();
	return 0;
}
```

