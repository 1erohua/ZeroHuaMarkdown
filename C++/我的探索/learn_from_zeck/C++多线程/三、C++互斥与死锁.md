# std的mutex们
在C++中，`std::mutex`、`std::recursive_mutex`、`std::timed_mutex` 和 `std::recursive_timed_mutex` 是用于多线程编程的同步原语。它们的主要目的是保护共享资源，防止多个线程同时访问或修改这些资源，从而避免数据竞争和不一致性问题。下面详细讲解它们的区别、效果、实现原理、本质、用法及用途。

### 1. `std::mutex`
#### 本质与实现原理：
`std::mutex` 是一个互斥锁（Mutex，Mutual Exclusion），它提供了最基本的线程同步机制。当一个线程锁定了 `std::mutex`，其他线程试图锁定它时会被阻塞，直到锁被释放。

#### 用法：
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void print_block(int n, char c) {
    mtx.lock();
    for (int i = 0; i < n; ++i) { std::cout << c; }
    std::cout << '\n';
    mtx.unlock();
}

int main() {
    std::thread th1(print_block, 50, '*');
    std::thread th2(print_block, 50, '$');

    th1.join();
    th2.join();

    return 0;
}
```
#### 效果：
- 确保同一时间只有一个线程可以执行 `print_block` 函数中的代码。
- 其他线程在 `mtx.lock()` 处被阻塞，直到当前线程释放锁。

#### 用途：
- 保护共享资源，避免数据竞争。
- 适用于简单的互斥场景。

### 2. `std::recursive_mutex`
#### 本质与实现原理：
`std::recursive_mutex` 是一个递归互斥锁，允许同一个线程多次锁定同一个互斥锁。每次锁定必须对应一次解锁，只有当锁的计数归零时，其他线程才能锁定它。

#### 用法：
```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::recursive_mutex mtx;

void foo(int n) {
    mtx.lock();
    std::cout << "Locked by thread " << n << '\n';
    if (n > 0) {
	    // 如果只是使用简单的std::mutex,只能在这里解锁了，解锁后资源还可能被抢
        foo(n - 1);
    }
    mtx.unlock();
}

int main() {
    std::thread th1(foo, 3);
    std::thread th2(foo, 2);

    th1.join();
    th2.join();

    return 0;
}
```
#### 效果：
- 允许同一个线程多次锁定同一个互斥锁。
- 其他线程在锁被完全释放之前无法锁定它。

**注**
> **不能直接使用 `std::mutex` 进行递归操作。**
> 
`std::mutex` 是一个非递归锁（non-recursive mutex），这意味着如果一个线程已经锁定了 `std::mutex`，再次尝试锁定它会导致**未定义行为**（通常是死锁或程序崩溃）。这是因为 `std::mutex` 不会记录锁的持有者，也无法区分是同一个线程还是不同线程尝试锁定它

#### 用途：
- 适用于递归函数或需要多次锁定同一互斥锁的场景。
- 避免死锁，尤其是在递归调用中。

### 3. `std::timed_mutex`
#### 本质与实现原理：
`std::timed_mutex` 是一个带超时功能的互斥锁。它允许线程尝试锁定互斥锁一段时间，如果在指定时间内未能锁定，线程可以选择继续执行其他操作而不是无限期等待。

#### 用法：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::timed_mutex mtx;

void foo() {
    auto now = std::chrono::steady_clock::now();
    if (mtx.try_lock_until(now + std::chrono::seconds(1))) {
        std::cout << "Locked by thread " << std::this_thread::get_id() << '\n';
        std::this_thread::sleep_for(std::chrono::seconds(2));
        mtx.unlock();
    } else {
        std::cout << "Failed to lock by thread " << std::this_thread::get_id() << '\n';
    }
}

int main() {
    std::thread th1(foo);
    std::thread th2(foo);

    th1.join();
    th2.join();

    return 0;
}
```
#### 效果：
- 线程尝试在指定时间内锁定互斥锁。
- 如果超时未能锁定，线程可以继续执行其他操作。

#### 用途：
- 适用于需要避免长时间阻塞的场景。
- 可以用于实现超时机制，避免死锁。

### 4. `std::recursive_timed_mutex`
#### 本质与实现原理：
`std::recursive_timed_mutex` 是 `std::recursive_mutex` 和 `std::timed_mutex` 的结合体。它既允许同一个线程多次锁定同一个互斥锁，也支持超时机制。

#### 用法：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <chrono>

std::recursive_timed_mutex mtx;

void foo(int n) {
    auto now = std::chrono::steady_clock::now();
    if (mtx.try_lock_until(now + std::chrono::seconds(1))) {
        std::cout << "Locked by thread " << std::this_thread::get_id() << '\n';
        if (n > 0) {
            foo(n - 1);
        }
        mtx.unlock();
    } else {
        std::cout << "Failed to lock by thread " << std::this_thread::get_id() << '\n';
    }
}

int main() {
    std::thread th1(foo, 3);
    std::thread th2(foo, 2);

    th1.join();
    th2.join();

    return 0;
}
```
#### 效果：
- 允许同一个线程多次锁定同一个互斥锁。
- 支持超时机制，避免长时间阻塞。

#### 用途：
- 适用于需要递归锁定且需要超时机制的场景。
- 结合了 `std::recursive_mutex` 和 `std::timed_mutex` 的优点。

### 总结
- **`std::mutex`**：最基本的互斥锁，适用于简单的互斥场景。
- **`std::recursive_mutex`**：允许同一个线程多次锁定同一个互斥锁，适用于递归函数或需要多次锁定同一互斥锁的场景。
- **`std::timed_mutex`**：支持超时机制，适用于需要避免长时间阻塞的场景。
- **`std::recursive_timed_mutex`**：结合了递归锁定和超时机制，适用于需要递归锁定且需要超时机制的场景。

这些互斥锁的选择取决于具体的应用场景和需求。正确使用它们可以有效地避免多线程编程中的竞争条件和死锁问题。


# 潜在的数据安全与线程安全问题
### 最初的问题
首先，先看下面定义的线程栈：
```cpp
template<typename T>
class threadsafe_stack1
{
private:
    std::stack<T> data;
    mutable std::mutex m;
public:
    threadsafe_stack1() {}
    threadsafe_stack1(const threadsafe_stack1& other)
    {
        std::lock_guard<std::mutex> lock(other.m);
        // ①在构造函数的函数体（constructor body）内进行复制操作
        data = other.data;
    }
    threadsafe_stack1& operator=(const threadsafe_stack1&) = delete;
    void push(T new_value)
    {
        std::lock_guard<std::mutex> lock(m);
        data.push(std::move(new_value));
    }

    // 这几段代码看起来也正常，要弹出，先调用判断空，再调用弹出就行了
    // 然而实际总是不如人意
    T pop()
    {
        std::lock_guard<std::mutex> lock(m);
        auto element = data.top();
        data.pop();
        return element;
    }

    bool empty() const
    {
        std::lock_guard<std::mutex> lock(m);
        return data.empty();
    }
};

```
实际的使用：
```cpp
void test_threadsafe_stack1() {
    threadsafe_stack1<int> safe_stack;
    safe_stack.push(1);

	// 观察这两个线程，我举例说明问题
	// 假设栈内目前只有一个元素
	// t1快于t2，t1先一步判断了栈不为空，进入if块
	// 但t1因为一些原因卡在内部，花了些时间
	// 而在t1卡在这个“花了些时间”的过程中，t2也进入了if判断
	// 并且由于在t2判断时，t1仍然在等待，并没有真正的弹出，因而栈也不为空
	// 最终t2也进入了if块内
	// 最终栈内会接连弹出两次
    std::thread t1([&safe_stack]() {
        if (!safe_stack.empty()) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            safe_stack.pop();
            }
        });

    std::thread t2([&safe_stack]() {
        if (!safe_stack.empty()) {
            std::this_thread::sleep_for(std::chrono::seconds(1));
            safe_stack.pop();
        }
    });

    t1.join();
    t2.join();
}
```
为什么会有这样的问题？ 个人理解，**pop和empty各自本身是线程安全的，就是单拎出来，是没有问题的。但是，线程安全的是它们函数的内部，而不是函数的返回值**。这里引用作者的一句话就是：
> *但是对于读取类型的操作，即使读取函数是线程安全的，但是返回值抛给外边使用，存在不安全性*。

就比如这里外部的操作不是线程安全的，产生了线程的竞争

下面是ai的说法，我会补充其中

### 1. 线程安全的基本概念
线程安全是指在多线程环境下，多个线程同时访问共享资源时，不会出现数据竞争或不一致的情况。为了保证线程安全，通常需要使用同步机制，如互斥锁（`std::mutex`）、条件变量（`std::condition_variable`）等。

### 2. 问题分析
在你提供的代码中，`threadsafe_stack1` 类试图通过互斥锁来保证栈的线程安全。然而，`pop` 操作存在潜在的线程安全问题。具体来说，`empty()` 和 `pop()` 是两个独立的操作，尽管它们各自是线程安全的，但在多线程环境下，两个线程可能同时检查栈是否为空，然后同时执行 `pop` 操作，导致数据竞争或崩溃。

### 3. 解决方案
#### 3.1 抛出异常
你提到的第一种解决方案是通过抛出异常来处理空栈的情况。这种方法的优点是可以在栈为空时明确地通知调用者，缺点是需要在调用处捕获异常，增加了代码的复杂性。

```cpp
T pop()
{
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) throw empty_stack();
    auto element = data.top();
    data.pop();
    return element;
}
```

#### 3.2 返回非法值
你提到的第二种解决方案是在栈为空时返回一个非法值。这种方法的优点是避免了异常处理的复杂性，缺点是对于复杂类型（如 `std::vector<int>`），很难定义一个合适的非法值。

```cpp
T pop()
{
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) return T(); // 假设 T 有一个默认构造函数
    auto element = data.top();
    data.pop();
    return element;
}
```

### 4. 更优化的方案
#### 4.1 引用传递
你提到的第四种解决方案是通过引用传递来返回栈顶元素。这种方法的优点是避免了数据拷贝，缺点是需要在调用处预先定义一个变量来接收结果。

```cpp
void pop(T& value)
{
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) throw empty_stack();
    value = data.top();
    data.pop();
}
```

#### 使用智能指针
为了避免 `empty()` 和 `pop()` 之间的竞争条件，可以将这两个操作合并为一个原子操作。例如，可以在 `pop()` 函数内部再次检查栈是否为空。
使用智能指针（`std::shared_ptr`）来返回栈顶元素。这种方法的优点是避免了数据拷贝，减少了内存溢出的风险，同时可以通过返回 `nullptr` 来处理空栈的情况。

```cpp
std::shared_ptr<T> pop()
{
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) return nullptr;
    std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
    data.pop();
    return res;
}
```
> 在 `std::shared_ptr<T> const res(std::make_shared<T>(data.top()));` 这一行中，`const` 关键字的作用是声明 `res` 为一个常量指针，即 `res` 本身不能被修改（例如，不能指向其他对象），但它指向的对象（即 `T` 类型的对象）是可以被修改的。
> 
>   - 使用 `const` 可以确保 `res` 在初始化后不会被重新赋值或修改。这可以避免在函数后续的逻辑中不小心修改 `res`，从而引入潜在的 bug。
>   
>   - 例如，如果没有 `const`，可能会不小心写出 `res = std::make_shared<T>(some_other_value);`，这会改变 `res` 的指向，导致逻辑错误。
>   
>   - 使用 `const` 可以明确表达设计意图：`res` 是一个只读的指针，它的值在初始化后不会改变。这有助于提高代码的可读性和可维护性。
> 
>   - 编译器可能会对 `const` 变量进行一些优化，因为它知道这些变量的值不会改变。
 
### 5. 内存管理
你提到的内存管理问题，特别是在 `pop` 操作中，如果 `T` 是一个复杂类型（如 `std::vector<int>`），在返回时可能会发生内存不足的情况。使用智能指针可以有效地减少内存拷贝，降低内存溢出的风险。

### 6. 总结
- **线程安全**：在多线程环境下，必须使用同步机制来保证共享资源的线程安全。
- **异常处理**：抛出异常是一种明确处理错误的方式，但会增加代码复杂性。
- **返回非法值**：对于简单类型，返回非法值是一种简单有效的处理方式，但对于复杂类型则不适用。
- **智能指针**：使用智能指针可以避免数据拷贝，减少内存溢出的风险，同时可以通过返回 `nullptr` 来处理空栈的情况。
- **引用传递**：通过引用传递可以避免数据拷贝，但需要在调用处预先定义变量。
- **组合操作**：将多个操作合并为一个原子操作，可以避免竞争条件。

### 7. 拓展
在实际应用中，除了使用互斥锁，还可以考虑使用其他同步机制，如读写锁（`std::shared_mutex`）、无锁数据结构等，以提高并发性能。此外，C++17 引入了 `std::optional`，可以用于表示可能不存在的值，进一步简化了空栈处理的逻辑。

```cpp
std::optional<T> pop()
{
    std::lock_guard<std::mutex> lock(m);
    if (data.empty()) return std::nullopt;
    auto element = data.top();
    data.pop();
    return element;
}
```

通过使用 `std::optional`，可以在栈为空时返回 `std::nullopt`，避免了抛出异常或返回非法值的复杂性。


# 同时加锁

### 同时加锁问题
当我们无法避免在一个函数内部使用两个互斥量，并且都要解锁的情况，那我们可以采取同时加锁的方式。我们先定义一个类,假设这个类不推荐拷贝构造，但我们也提供了这个类的拷贝构造和移动构造

```cpp
class som_big_object {
public:
    som_big_object(int data) :_data(data) {}
    //拷贝构造
    som_big_object(const som_big_object& b2) :_data(b2._data) {
        _data = b2._data;
    }
    //移动构造
    som_big_object(som_big_object&& b2) :_data(std::move(b2._data)) {

    }
    //重载输出运算符
    friend std::ostream& operator << (std::ostream& os, const som_big_object& big_obj) {
        os << big_obj._data;
        return os;
    }

    //重载赋值运算符
    som_big_object& operator = (const som_big_object& b2) {
        if (this == &b2) {
            return *this;
        }
        _data = b2._data;
        return *this;
    }

    //交换数据
    friend void swap(som_big_object& b1, som_big_object& b2) {
        som_big_object temp = std::move(b1);
        b1 = std::move(b2);
        b2 = std::move(temp);
    }
private:
    int _data;
};
```

接下来我们定义一个类对上面的类做管理，为防止多线程情况下数据混乱， 包含了一个互斥量。

```cpp
class big_object_mgr {
public:
    big_object_mgr(int data = 0) :_obj(data) {}
    void printinfo() {
        std::cout << "current obj data is " << _obj << std::endl;
    }
    friend void danger_swap(big_object_mgr& objm1, big_object_mgr& objm2);
    friend void safe_swap(big_object_mgr& objm1, big_object_mgr& objm2);
    friend void safe_swap_scope(big_object_mgr& objm1, big_object_mgr& objm2);
private:
    std::mutex _mtx;
    som_big_object _obj;
};
```

下面的问题其实还是经典的一个线程先锁1再锁2，另一个线程先锁2再锁1

```cpp
void danger_swap(big_object_mgr& objm1, big_object_mgr& objm2) {
    std::cout << "thread [ " << std::this_thread::get_id() << " ] begin" << std::endl;
    if (&objm1 == &objm2) {
        return;
    }
    std::lock_guard <std::mutex> gurad1(objm1._mtx);
    //此处为了故意制造死锁，我们让线程小睡一会
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::lock_guard<std::mutex> guard2(objm2._mtx);
    swap(objm1._obj, objm2._obj);
    std::cout << "thread [ " << std::this_thread::get_id() << " ] end" << std::endl;
}
```

`danger_swap`是危险的交换方式。比如如下调用

```cpp
void  test_danger_swap() {
    big_object_mgr objm1(5);
    big_object_mgr objm2(100);

    std::thread t1(danger_swap, std::ref(objm1), std::ref(objm2));
    std::thread t2(danger_swap, std::ref(objm2), std::ref(objm1));
    t1.join();
    t2.join();

    objm1.printinfo();
    objm2.printinfo();
}
```

这种调用方式存在隐患，因为`danger_swap`函数在两个线程中使用会造成互相竞争加锁的情况。 那就需要用锁同时锁住两个锁。

**这里的改进在于，锁的顺序已经由std::lock进行管理了，而std::adopt_lock使得std::lock_guard所干的，就是领养。**
> `std::lock` 不仅仅是对锁进行排序，它还会在排序的同时对传入的互斥量进行加锁操作。而 `std::lock_guard` 的作用是管理这些已经加锁的互斥量，确保它们在作用域结束时自动释放。
```cpp
void safe_swap(big_object_mgr& objm1, big_object_mgr& objm2) {
    std::cout << "thread [ " << std::this_thread::get_id() << " ] begin" << std::endl;
    if (&objm1 == &objm2) {
        return;
    }

    std::lock(objm1._mtx, objm2._mtx);
    //领养锁管理它自动释放
    std::lock_guard <std::mutex> gurad1(objm1._mtx, std::adopt_lock);

    //此处为了故意制造死锁，我们让线程小睡一会
    std::this_thread::sleep_for(std::chrono::seconds(1));

    std::lock_guard <std::mutex> gurad2(objm2._mtx, std::adopt_lock);

    swap(objm1._obj, objm2._obj);
    std::cout << "thread [ " << std::this_thread::get_id() << " ] end" << std::endl;
}
```

比如下面的调用就是合理的

```cpp
void test_safe_swap() {
    big_object_mgr objm1(5);
    big_object_mgr objm2(100);

    std::thread t1(safe_swap, std::ref(objm1), std::ref(objm2));
    std::thread t2(safe_swap, std::ref(objm2), std::ref(objm1));
    t1.join();
    t2.join();

    objm1.printinfo();
    objm2.printinfo();
}
```

在 `danger_swap` 函数中，两个互斥量 `objm1._mtx` 和 `objm2._mtx` 是分别加锁的。这种方式在多线程环境下可能会导致死锁。例如，当两个线程同时调用 `danger_swap` 时，一个线程可能先锁住 `objm1._mtx`，而另一个线程先锁住 `objm2._mtx`，然后两个线程都会尝试去锁住对方已经锁住的互斥量，从而导致死锁。

为了避免这种情况，`safe_swap` 函数使用了 `std::lock` 函数来同时锁住两个互斥量。`std::lock` 是一个特殊的函数，它可以同时锁住多个互斥量，并且不会产生死锁。具体来说，`std::lock` 会以一种避免死锁的方式依次锁住所有传入的互斥量。

### `safe_swap` 函数的改动：
1. **同时加锁**：
   ```cpp
   std::lock(objm1._mtx, objm2._mtx);
   ```
   这行代码同时锁住了 `objm1._mtx` 和 `objm2._mtx`，避免了分别加锁可能导致的死锁问题。

2. **领养锁**：
   ```cpp
   std::lock_guard<std::mutex> gurad1(objm1._mtx, std::adopt_lock);
   std::lock_guard<std::mutex> gurad2(objm2._mtx, std::adopt_lock);
   ```
   在 `std::lock` 锁住互斥量之后，使用 `std::lock_guard` 来管理这些锁。`std::adopt_lock` 参数告诉 `std::lock_guard` 对象，它不需要再次锁住互斥量，而是直接“领养”已经锁住的互斥量，并在 `std::lock_guard` 对象销毁时自动释放锁。

### 为什么这些改动是合理的：
- **避免死锁**：`std::lock` 保证了同时锁住多个互斥量时不会产生死锁。
- **自动释放锁**：`std::lock_guard` 确保了在函数结束时，互斥量会被自动释放，避免了手动释放锁的麻烦和潜在的错误。
- **代码简洁**：使用 `std::lock` 和 `std::lock_guard` 使得代码更加简洁和易于维护。

### 示例调用：
```cpp
void test_safe_swap() {
    big_object_mgr objm1(5);
    big_object_mgr objm2(100);

    std::thread t1(safe_swap, std::ref(objm1), std::ref(objm2));
    std::thread t2(safe_swap, std::ref(objm2), std::ref(objm1));
    t1.join();
    t2.join();

    objm1.printinfo();
    objm2.printinfo();
}
```
在这个调用中，`safe_swap` 函数确保了在多线程环境下安全地交换两个 `big_object_mgr` 对象的数据，而不会产生死锁。

总结来说，`safe_swap` 函数通过使用 `std::lock` 和 `std::lock_guard` 来同时加锁和管理锁，避免了多线程环境下的死锁问题，确保了线程安全。


# 层级锁
```cpp
//层级锁
class hierarchical_mutex {
public:
	// 创建给一个初始锁
    explicit hierarchical_mutex(unsigned long value) :_hierarchy_value(value),
        _previous_hierarchy_value(0) {}

	// 删除拷贝构造与赋值
    hierarchical_mutex(const hierarchical_mutex&) = delete;
    hierarchical_mutex& operator=(const hierarchical_mutex&) = delete;
    
    void lock() {
        check_for_hierarchy_violation();
        _internal_mutex.lock();
        update_hierarchy_value();
    }

    void unlock() {
        if (_this_thread_hierarchy_value != _hierarchy_value) {
            throw std::logic_error("mutex hierarchy violated");
        }

        _this_thread_hierarchy_value = _previous_hierarchy_value;
        _internal_mutex.unlock();
    }

    bool try_lock() {
        check_for_hierarchy_violation();
        // 这里调用的是std::mutex的try_lock方法
        if (!_internal_mutex.try_lock()) {
            return false;
        }

        update_hierarchy_value();
        return true;
    }
private:
    std::mutex  _internal_mutex;

	// 一个线程会有多个锁，每个锁有一个初始化之后固定的层级_hierarchy_value
	// 一个线程会有一个实时更新的 _this_thread_hierarchy_value
	// 由于层级锁的强制要求，因而会使得每次加锁时，层级会提升（即层级值会减少）
	// 因而线程层级锁记录的是最高级的层级
	// 每次加锁都要看这个层级进行比对，只有更高的层级才能创建锁

    // 当前层级值
    // 当前锁的层级，一旦确定就不可变
    unsigned long const _hierarchy_value;
    
    // 上一次层级值
    unsigned long _previous_hierarchy_value;
    
    // 本线程记录的层级值
    static thread_local  unsigned long  _this_thread_hierarchy_value;

	// 层级违规
    void check_for_hierarchy_violation() {
        if (_this_thread_hierarchy_value <= _hierarchy_value) {
            throw  std::logic_error("mutex  hierarchy violated");
        }
    }

    void  update_hierarchy_value() {
        _previous_hierarchy_value = _this_thread_hierarchy_value;
        _this_thread_hierarchy_value = _hierarchy_value;
    }
};

thread_local unsigned long hierarchical_mutex::_this_thread_hierarchy_value(ULONG_MAX);

void test_hierarchy_lock() {
    hierarchical_mutex  hmtx1(1000);
    hierarchical_mutex  hmtx2(500);
    std::thread t1([&hmtx1, &hmtx2]() {
        hmtx1.lock();
        hmtx2.lock();
        hmtx2.unlock();
        hmtx1.unlock();
        });

    std::thread t2([&hmtx1, &hmtx2]() {
        hmtx2.lock();
        hmtx1.lock();
        hmtx1.unlock();
        hmtx2.unlock();
        });

    t1.join();
    t2.join();
}

```


#  std::mutex::try_lock
上面的代码有用到这个，稍微讲讲：

`std::mutex::try_lock` 是 C++ 标准库中 `std::mutex` 类的一个成员函数，用于尝试获取互斥锁的所有权。与 `lock` 方法不同，`try_lock` 是非阻塞的，它会立即返回，而不是等待锁变为可用。

### 函数原型
```cpp
bool try_lock();
```

### 返回值
- **成功获取锁**：返回 `true`。
- **未能获取锁**：返回 `false`。

### 使用场景
`try_lock` 适用于以下场景：
1. **非阻塞操作**：当你不想让线程阻塞等待锁时，可以使用 `try_lock` 来尝试获取锁。如果锁不可用，线程可以继续执行其他任务。
2. **避免死锁**：在某些情况下，使用 `try_lock` 可以帮助避免死锁，尤其是在需要获取多个锁时。

### 示例代码
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mtx;
int shared_data = 0;

void increment() {
    for (int i = 0; i < 10000; ++i) {
        if (mtx.try_lock()) {  // 尝试获取锁
            ++shared_data;
            mtx.unlock();      // 释放锁
        } else {
            // 未能获取锁，执行其他操作
            std::this_thread::yield();
        }
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Final shared_data value: " << shared_data << std::endl;
    return 0;
}
```

### 注意事项
1. **锁的释放**：成功获取锁后，必须手动调用 `unlock` 来释放锁，否则会导致死锁。
2. **性能考虑**：在高竞争环境下，频繁使用 `try_lock` 可能会导致性能下降，因为它需要不断尝试获取锁。
3. **异常安全**：如果 `try_lock` 成功获取锁，但在后续代码中抛出异常，可能会导致锁未被释放。可以使用 `std::lock_guard` 或 `std::unique_lock` 来管理锁的生命周期，确保异常安全。

### 总结
`std::mutex::try_lock` 是一个非常有用的工具，特别是在需要非阻塞锁操作的场景中。然而，使用时需要注意锁的管理和性能影响。