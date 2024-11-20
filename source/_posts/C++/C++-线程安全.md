---
title: C++-线程安全
tags: C++
category: C++
abbrlink: b0174cac
date: 2024-08-01 11:42:34
---
[参考博客](http://www.seestudy.cn/?list_9/35.html)

# thread
## int
```C++
#include<iostream>
#include<thread>
#include<string>
#include<memory>
using namespace std;
void func(int& x)
{
	x += 1;
}

int main()
{
	int a = 1;
	thread t1(func, ref(a));
	t1.join();
	cout << a << endl;
	return 0;
}
```
## 类
```C++
#include<iostream>
#include<thread>
#include<string>
#include<memory>
using namespace std;

class A {
public:
	void func() {
		cout << "Hello !" << endl;
	}
};



int main()
{
    // 智能指针 当当前变量不再被需要时 自动释放
	shared_ptr<A> a = make_shared<A>();

	thread t(&A::func, a);
	t.join();

	return 0;
}

```

# mutex

```C++
#include<iostream>
#include<thread>
#include<mutex>

using namespace std;
timed_mutex mtx;

void func(int& x) {
	for (int i = 0; i < 2; i++) {
		unique_lock<timed_mutex> lg(mtx, defer_lock);
		if (lg.try_lock_for(chrono::seconds(2))) {
			x++;
			this_thread::sleep_for(chrono::seconds(1));
		}
	}
}

int main()
{
	int n = 0;
	thread thread1(func, ref(n));
	thread thread2(func, ref(n));
	thread1.join();
	thread2.join();
	cout << n << endl;
	return 0;
}
```


# unique_lock shard_lock

# condition_variable
## 生产者 消费者
```C++
#include<iostream>
#include<thread>
#include<mutex>
#include<queue>
#include<condition_variable>
using namespace std;

mutex mtx;
queue<int> mque;
condition_variable cg;

void productor() {
	for (int i = 0; i < 100; i++) {
		unique_lock<mutex> ul(mtx);
		mque.push(i);
		cg.notify_one();
		cout << "生产了" << i << endl;
		
		this_thread::sleep_for(chrono::microseconds(100));
	}
}

void consumer() {
	while (1) {
		unique_lock<mutex> ul(mtx);
		cg.wait(ul, []() {return !mque.empty(); });

		int value = mque.front();
		mque.pop();
		cout << "消费了"<<value<<endl;
	}

}

int main()
{
	thread prod(productor);
	thread cons(consumer);

	prod.join();
	cons.join();

	return 0;
}
```

# atomic 
原子操作 比 加锁 效率高

```C++
#include<iostream>
#include<thread>
#include<mutex>
using namespace std;

atomic<int> shared_data = 0;

void func() {
	for (int i = 0; i < 10000000; i++) {
		shared_data++;
	}
}

int main()
{
	thread t1(func);
	thread t2(func);
	t1.join();
	t2.join();
	//store load 操作 可以保证原子性
	shared_data.store(1);
	cout << shared_data.load() << endl;
	return 0;
}
```