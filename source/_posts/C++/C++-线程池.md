---
title: C++-线程池
date: 2024-09-01 10:31:54
tags: C++
category: C++
---
# 涉及知识点

- mutex
- lock
- lamda表达式
- 队列queue
- emplace
- 可变参数
- 模板函数
- 万能引用
- 左值引用
- 右值引用
- 完美转发forward
- 函数适配器bind
- 时间库chrono

# 代码

```C++
#include<iostream>
#include<thread>
#include<mutex>
#include<condition_variable>
#include<string>
#include<queue>
#include<vector>
#include<functional>
using namespace std;

class ThreadPool {
public:
	ThreadPool(int numThreads): stop(false) {
		for (int i = 0; i < numThreads; i++) {
			threads.emplace_back([this] {
				while (1) {
					function<void()> task;
					{
						unique_lock<mutex> lock(mtx);
						condition.wait(lock, [this] {
							return !tasks.empty() || stop;
							});
						if (stop && tasks.empty()) {
							return;
						}
						task = move(tasks.front());
						tasks.pop();
					}
					task();
				}
			});
		}
	}

	~ThreadPool() {
		{
			unique_lock<mutex> lock(mtx);
			stop = true;
		}
		condition.notify_all();
		for (auto& t : threads) {
			t.join();
		}
	}
	template<class F, class... Args>
	void enqueue(F && f, Args&&... args) {
		function<void()>task = bind(forward<F>(f), forward<Args>(args)...);
		{
			unique_lock<mutex> lock(mtx);
			tasks.emplace(move(task));
		}
		condition.notify_one();
	}
private:
	vector<thread> threads;
	queue<function<void()> > tasks;

	mutex mtx;
	condition_variable condition;

	bool stop;
};


int main()
{
	ThreadPool pool(4);

	for (int i = 0; i < 12; i++) {
		pool.enqueue([i] {
			cout << "task : " << i << "is running !"<<endl;
			this_thread::sleep_for(chrono::seconds(2));
			cout << "task : " << i << "is close !" << endl;
			});
	}

	return 0;
}
```