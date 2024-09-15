---
title: C++-异步并发
date: 2024-09-01 11:15:42
tags: C++
category: C++
---

# async

```C++
#include<iostream>
#include<future>
#include<thread>
using namespace std;

int func() {
	int ret = 0;
	for (int i = 0; i < 1000; i++) {
		//cout << "func " << i << endl;
		ret++;
	}
	return ret;
}

int main() {
	// async 自动在后台开启线程
	future<int> future_ret = async(launch::async, func);
	cout << func() << endl;

	cout << future_ret.get() << endl;
	return 0;
}

```

# packaged_task

```C++
#include<iostream>
#include<future>
#include<thread>
using namespace std;

int func() {
	int ret = 0;
	for (int i = 0; i < 1000; i++) {
		//cout << "func " << i << endl;
		ret++;
	}
	return ret;
}

int main() {
	// packaged 不会自动在后台开启线程
	packaged_task<int()> task(func);
	future<int> future_ret = task.get_future();

	thread t1(move(task));

	t1.join();
	cout << future_ret.get() << endl;

	return 0;
}

```


# promise
## 介绍promise
`std::promise` 是 C++ 标准库中的一个类，用于在多线程环境中实现异步操作和线程间的同步。它是 <future> 头文件的一部分，与 std::future 配合使用，提供了一个机制来在线程之间传递结果或异常。

### 主要概念
- std::promise：用于设置一个值或异常，这个值或异常会被 std::future 读取。std::promise 可以被看作是生产者，它负责提供值或异常。
- std::future：用于获取 std::promise 设置的值或异常。std::future 可以被看作是消费者，它负责接收和处理值或异常。
### 典型用法
std::promise 和 std::future 的常见使用场景包括：线程之间的结果传递、异步操作的实现、以及与 std::async 结合使用等。

### 示例代码
以下是一个示例，演示了如何使用 std::promise 和 std::future：
```C++
#include <iostream>
#include <thread>
#include <future>
#include <exception>

// 线程函数
void compute(std::promise<int>&& promise) {
    try {
        // 进行计算
        int result = 42; // 模拟计算结果
        // 设置值
        promise.set_value(result);
    } catch (...) {
        // 捕获异常并设置异常
        promise.set_exception(std::current_exception());
    }
}

int main() {
    // 创建 promise 和 future 对象
    std::promise<int> promise;
    std::future<int> future = promise.get_future();

    // 创建线程，传递 promise
    std::thread t(compute, std::move(promise));

    try {
        // 等待计算结果并获取值
        int result = future.get();
        std::cout << "Result: " << result << std::endl; // 输出 42
    } catch (const std::exception& e) {
        std::cout << "Exception: " << e.what() << std::endl;
    }

    t.join(); // 等待线程结束
    return 0;
}

```

### promise 的主要操作
- `set_value(const T& value)`：设置成功的结果。
- `set_value(T&& value)`：设置成功的结果，接受右值。
- `set_exception(std::exception_ptr e)`：设置异常。
- `set_value_at_thread_exit(const T& value)`：设置线程结束时的值（此方法在 C++20 中引入）。

### 总结
`std::promise` 和 `std::future` 提供了一种简洁的方式来实现线程间的结果传递和同步。std::promise 用于设置值或异常，std::future 用于获取这些值或异常。它们可以帮助你构建更复杂的异步程序，并提供了一种安全的方式来处理线程间的通信。

## 问题一：
### Q
```C++
std::future<int> future = promise.get_future();
// 创建线程，传递 promise
std::thread t(compute, std::move(promise)); 
```
这里移动了promise 那么在主进程中的future 还能关联到promise吗，是不是关联到了移动到 子进程里的promise 

### A
是的，当你将 `promise` 移动到子线程中时，主线程中的 `future` 仍然与原来的 `promise` 关联。这是因为 `std::promise` 和 `std::future` 之间的关联关系是在调用 `promise.get_future()` 时就已经建立的，并且这种关联关系是持久的，不会因为 `promise` 对象的移动而改变。

#### 详细解释

1. **关联的建立**：
   - 当你调用 `promise.get_future()` 时，`std::future<int> future` 就与 `promise` 关联起来了。这意味着 `future` 会等待 `promise` 设置一个值或异常。

2. **`promise` 的移动**：
   - `std::promise` 可以被安全地移动，这是因为 `std::promise` 内部管理的资源（如与 `std::future` 的关联状态）并不会因为对象的移动而改变。
   - 当你调用 `std::move(promise)` 时，`promise` 对象的内部状态被转移到新对象中（即子线程中的 `promise` 参数），而原来的 `promise` 对象在主线程中就变成了一个空的（无效的）状态。

3. **`future` 的有效性**：
   - 即使 `promise` 被移动到子线程中，`future` 依然保持有效。它仍然会等待来自子线程中的 `promise` 设置的值或异常。
   - 移动 `promise` 只是将 `promise` 的所有权转移到另一个作用域（子线程），但与 `future` 的关联性保持不变。

4. **总结**：
   - 移动 `promise` 不会影响 `future` 的功能，`future` 依然可以从移动后的 `promise` 中获取结果或异常。因此，`future.get()` 在主线程中仍然能够正确地获取子线程中通过移动后的 `promise` 设置的结果。

换句话说，`std::promise` 和 `std::future` 之间的关联关系是基于共享状态的，这种状态是独立于 `promise` 本身的具体位置或所有权的，所以移动 `promise` 并不会破坏这个关联。

## 问题二
### Q
那么在子线程中promise结束被销毁后主线程的future也就不能用了吧?

### A
是的，在子线程中 `promise` 被销毁后，主线程中的 `future` 将无法再从这个 `promise` 获取到值。这是因为 `promise` 和 `future` 共享的状态一旦 `promise` 对象被销毁而没有设置任何值或异常，主线程中的 `future` 在访问时会抛出一个 `std::future_error` 异常。

#### 详细解释

1. **`promise` 和 `future` 的共享状态**：
   - `std::promise` 和 `std::future` 共享一个内部状态，这个状态用于存储 `promise` 设置的值或异常。
   - 当 `promise` 设置了值（`set_value()`）或异常（`set_exception()`）时，`future` 可以通过 `get()` 方法访问这个状态。

2. **`promise` 的销毁**：
   - 如果 `promise` 在没有设置值或异常的情况下被销毁（即 `promise` 的析构函数被调用），那么 `future` 共享的状态会被标记为 "无效"。
   - 在这种情况下，如果主线程中的 `future` 调用 `get()` 方法，会抛出 `std::future_error` 异常，具体来说是 `std::future_errc::broken_promise`。

3. **`promise` 销毁的影响**：
   - 当 `promise` 被销毁后，主线程中的 `future` 就无法再获取到有效的结果。任何试图从这个 `future` 获取结果的操作都会导致异常。

4. **示例代码**：

```cpp
#include <iostream>
#include <thread>
#include <future>
#include <stdexcept>

void compute(std::promise<int>&& promise) {
    // 不设置值，直接销毁 promise
}

int main() {
    std::promise<int> promise;
    std::future<int> future = promise.get_future();

    std::thread t(compute, std::move(promise));

    t.join(); // 等待线程结束

    try {
        int result = future.get(); // 这里会抛出异常，因为 promise 被销毁
    } catch (const std::future_error& e) {
        std::cout << "Exception: " << e.what() << std::endl;
    }

    return 0;
}
```

在上面的示例中，因为子线程中的 `promise` 被销毁而没有设置值或异常，主线程中的 `future.get()` 调用将会抛出一个 `std::future_error` 异常，表示 "broken promise"。

#### 总结

- `promise` 销毁后，如果没有设置任何值或异常，主线程中的 `future` 将无法正常工作，并且在访问时会抛出异常。
- 这意味着你需要确保 `promise` 在被销毁之前已经设置了一个值或异常，来保证 `future` 在主线程中的有效性。

## 问题三
### Q
如果设置了值并销毁，为什么future还能拿到，它不是以及销毁了吗?

### A
这是因为 `std::promise` 和 `std::future` 之间的共享状态是在堆上分配的，并且是独立于 `std::promise` 对象本身的生命周期的。

#### 详细解释

1. **共享状态的分离**：
   - 当你创建一个 `std::promise` 对象并调用 `get_future()` 时，`std::promise` 和 `std::future` 会共享一个内部状态（`shared state`），这个状态通常包括存储结果值、异常、以及同步机制的内容。
   - 这个共享状态是动态分配的（通常在堆上），而不是直接存储在 `std::promise` 或 `std::future` 对象中。因此，这个状态的生命周期独立于 `std::promise` 和 `std::future` 对象本身。

2. **`promise` 的作用**：
   - `std::promise` 的主要作用是通过 `set_value()` 或 `set_exception()` 来设置共享状态的内容。一旦这个值被设置，`promise` 对象的任务就基本完成了。
   - 设置完成后，即使 `std::promise` 对象被销毁，存储的结果（或异常）仍然存在于共享状态中，`std::future` 仍然能够访问它。

3. **`future` 的行为**：
   - 当你在主线程中使用 `future.get()` 时，它从共享状态中检索结果值或异常，而不是直接从 `promise` 对象中获取。
   - 因为共享状态在 `promise` 对象销毁后依然存在，所以 `future.get()` 可以在 `promise` 销毁后仍然正常工作。

#### 示例代码

```cpp
#include <iostream>
#include <thread>
#include <future>

void compute(std::promise<int>&& promise) {
    promise.set_value(42); // 设置值
    // promise 对象即将被销毁
}

int main() {
    std::promise<int> promise;
    std::future<int> future = promise.get_future();

    std::thread t(compute, std::move(promise));

    t.join(); // 等待线程结束

    // 即使 promise 对象已销毁，future 仍能从共享状态中获取值
    int result = future.get();
    std::cout << "Result: " << result << std::endl; // 输出：Result: 42

    return 0;
}
```

#### 总结

- 当 `std::promise` 设置了值后，它的内部共享状态将包含这个值，即使 `promise` 对象本身被销毁，这个值仍然存在于共享状态中。
- `std::future` 在访问结果时，实际是从共享状态中获取数据，而不是直接依赖于 `promise` 对象的存在。这就是为什么即使 `promise` 被销毁了，`future` 仍然能够获取到先前设置的值。
