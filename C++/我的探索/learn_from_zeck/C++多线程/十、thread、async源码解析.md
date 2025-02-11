# thread参数拷贝
功力提升了，回看一遍

```cpp
template <class _Fn, class... _Args, enable_if_t<!is_same_v<_Remove_cvref_t<_Fn>, thread>, int> = 0>
    _NODISCARD_CTOR explicit thread(_Fn&& _Fx, _Args&&... _Ax) {
        _Start(_STD forward<_Fn>(_Fx), _STD forward<_Args>(_Ax)...);
    }
```
完美转发+万能引用，完全保留 **传递的类型**。
然后在构造函数内调用_Start函数，将接受的值完美转发到该函数

```cpp
    template <class _Fn, class... _Args>
    void _Start(_Fn&& _Fx, _Args&&... _Ax) {
        // 1 处
        using _Tuple                 = tuple<decay_t<_Fn>, decay_t<_Args>...>;
        // 2 处
        auto _Decay_copied           = _STD make_unique<_Tuple>(_STD forward<_Fn>(_Fx), _STD forward<_Args>(_Ax)...); // 将函数与值，按照值拷贝，放进tuple，同时获得一个孤独指针
        // 3 处
        constexpr auto _Invoker_proc = _Get_invoke<_Tuple>(make_index_sequence<1 + sizeof...(_Args)>{});

#pragma warning(push)
#pragma warning(disable : 5039) // pointer or reference to potentially throwing function passed to
                                // extern C function under -EHc. Undefined behavior may occur
                                // if this function throws an exception. (/Wall)
        // 4处
        _Thr._Hnd =
            reinterpret_cast<void*>(_CSTD _beginthreadex(nullptr, 0, _Invoker_proc, _Decay_copied.get(), 0, &_Thr._Id));
#pragma warning(pop)

        if (_Thr._Hnd) { // ownership transferred to the thread
            (void) _Decay_copied.release();
        } else { // failed to start thread
            _Thr._Id = 0;
            _Throw_Cpp_error(_RESOURCE_UNAVAILABLE_TRY_AGAIN);
        }
    }
```
以下皆是原文解释

我们从上面的代码 1处 可以看到`_Tuple`是一个去引用的类型，因为其内部存储的都是`decay_t`过后的类型，所以无论左值引用还是右值引用到这里都变为去引用的类型。
> 关于decay_t: [[std的decay_t——类型退化]]

所以2处就是将参数和函数按照值拷贝的方式存在tuple中。

3处定义了一个可调用对象`_Invoker_proc`

4处启动线程调用`_Invoker_proc`进而调用我们传递的回调函数和参数。

所以综上所述，`std::thread`向回调函数传递值是以副本的方式,回调函数参数是引用类型，可以将传递的实参用`std::ref`包装达到修改的效果。 因为`std::ref`其实是构造了`reference_wrapper`类对象，这个类实现了仿函数

```cpp
_CONSTEXPR20 operator _Ty&() const noexcept {
        return *_Ptr;
}
```
所以当线程接收`std::ref`包裹的参数时会调用仿函数通过指针解引用的方式获取外部实参，以`_Ty&`返回，从而达到修改的效果。

[[std的ref]]

关于AI的解释
你提到的内容涉及 `std::thread` 的参数传递机制和 `std::ref` 的作用原理。以下是对原文逐段的解释和总结，结合代码示例帮助你理解：

---

### **1. `std::thread` 的参数存储机制**
当创建 `std::thread` 对象时，线程的构造函数会将传递的参数（包括回调函数和参数）存储到内部的 `tuple` 中。存储前会对每个参数进行 `std::decay_t` 处理，目的是：
- 移除参数的引用（包括左值引用 `&` 和右值引用 `&&`）。
- 移除 `const`/`volatile` 限定符。
- 将数组退化为指针，函数退化为函数指针。

**代码示例**：
```cpp
void func(int& x) { x = 100; }

int main() {
    int x = 42;
    std::thread t(func, x);  // 参数 x 被 decay_t 处理为 int，按值拷贝存储
    t.join();
    // 此时 x 的值仍为 42，未被修改！
}
```
这里 `std::thread` 内部存储的是一个 `int` 的副本，而非引用，因此 `func` 无法修改原始的 `x`。

---

### **2. `std::ref` 的魔法**
如果希望线程修改外部变量，需要用 `std::ref` 包装参数：
```cpp
std::thread t(func, std::ref(x));  // 参数被包装为 reference_wrapper<int>
t.join();
// 此时 x 的值被修改为 100！
```

#### **为什么 `std::ref` 能绕过 `decay_t`？**
- `std::ref(x)` 返回一个 `std::reference_wrapper<T>` 对象（假设 `x` 的类型是 `T`）。
- `std::decay_t` 对 `std::reference_wrapper<T>` **不起作用**，因为：
  - `reference_wrapper` 是一个类类型，不是原生引用类型（如 `int&`）。
  - `std::decay_t` 只会移除原生引用、`const` 等修饰符，但不会改变 `reference_wrapper` 的类型。

因此，`std::thread` 内部存储的是 `reference_wrapper<int>` 对象（而非 `int` 的副本）。

---

### **3. `reference_wrapper` 的隐式转换**
`std::reference_wrapper<T>` 的关键在于它实现了隐式类型转换运算符：
```cpp
operator T&() const noexcept { return *_Ptr; }
```
这个运算符允许 `reference_wrapper<T>` 在需要 `T&` 的上下文中自动转换为原始类型的引用。

**线程调用的过程**：
1. `std::thread` 内部存储 `reference_wrapper<int>` 对象。
2. 当调用回调函数 `func` 时，需要传递 `int&` 类型的参数。
3. `reference_wrapper<int>` 会自动调用 `operator int&()`，返回其内部持有的原始对象（即 `x`）的引用。
4. 因此，`func` 实际操作的是原始的 `x`，而非副本。

---

### **4. 代码示例解析**
结合你提到的代码结构：
```cpp
// 伪代码表示线程内部存储和调用过程
template <class Fn, class... Args>
thread::thread(Fn&& fn, Args&&... args) {
    // 1. 对参数进行 decay_t 处理，移除引用等修饰符
    using DecayedArgs = std::tuple<std::decay_t<Args>...>;
    DecayedArgs _Tuple(std::forward<Args>(args)...);

    // 2. 存储回调函数和参数
    // 3. 创建可调用对象 _Invoker_proc，绑定 _Tuple
    // 4. 启动线程，调用 _Invoker_proc
}
```
- 如果传递 `std::ref(x)`，`std::decay_t` 处理后，`_Tuple` 中存储的是 `reference_wrapper<int>`。
- 当调用回调函数时，`reference_wrapper<int>` 会隐式转换为 `int&`，从而修改原始变量。

---

### **总结**
- **`std::decay_t` 的作用**：移除参数的引用和修饰符，导致 `std::thread` 默认按值存储参数。
- **`std::ref` 的机制**：
  - 将参数包装为 `reference_wrapper<T>`，避免 `decay_t` 的“去引用”效果。
  - 通过隐式转换运算符 `operator T&()`，在调用时还原为原始引用。
- **应用场景**：当需要线程修改外部变量时，必须用 `std::ref` 或 `std::cref` 包装参数。

---

### **对比实验**
```cpp
void func(int& x) { x = 100; }

int main() {
    int x = 42;
    // 实验1：直接传递 x（按值拷贝）
    std::thread t1(func, x);
    t1.join();
    std::cout << x << std::endl;  // 输出 42

    // 实验2：传递 std::ref(x)（按引用）
    std::thread t2(func, std::ref(x));
    t2.join();
    std::cout << x << std::endl;  // 输出 100
}
```

通过这种方式，可以直观理解 `std::ref` 如何绕过 `std::decay_t` 的影响，保持引用的语义。


原文：
那么如下调用就会报错，提示“invoke”: 未找到匹配的重载函数。

```cpp
void ChangeValue() {
    int m = 100;
    std::thread  t1{ [](int& rm) {
        rm++;
    }, m };

    t1.join();
}
```

因为 `invoke`函数调用时会将参数以右值的方式移动给回调函数，这会造成左值引用绑定右值的情况，所以编译报错。

改为下面这样写就没问题了

```cpp
void ChangeValue() {
    int m = 100;
    std::thread  t1{ [](int& rm) {
        rm++;
    }, std::ref(m) };

    t1.join();
}
```

# async
在C++中使用`std::async`时，由于其线程生命周期管理与JavaScript等单线程语言不同，需特别注意以下事项：

---

### **1. `std::future`的析构行为**
- **问题**：`std::async`返回的`std::future`析构时，若其是共享状态（`shared state`）的最后持有者，且任务未完成，析构会阻塞直到任务结束。
- **示例**：以下代码看似异步，实则同步执行：
  ```cpp
  void BlockAsync() {
      std::cout << "begin block async" << std::endl;
      {
          // future为右值，作用域结束析构时阻塞等待任务完成
          std::async(std::launch::async, []() { 
              std::this_thread::sleep_for(3s);
              std::cout << "async task done" << std::endl; 
          });
      }
      std::cout << "end block async" << std::endl;
  }
  ```
  **输出**：
  ```
  begin block async
  async task done
  end block async
  ```

---

### **2. 死锁陷阱**
- **场景**：主线程持有锁，异步任务尝试获取同一锁，而`future`析构时等待任务完成，导致死锁。
  ```cpp
  void DeadLock() {
      std::mutex mtx;
      std::lock_guard<std::mutex> lock(mtx);
      auto future = std::async(std::launch::async, [&mtx]() {
          std::lock_guard<std::mutex> lock(mtx); // 等待主线程释放锁
          // ... 
      });
      // future析构时阻塞，但主线程未释放锁，死锁！
  }
  ```
- **解决**：避免在异步任务中依赖主线程释放的资源，或确保`future`生命周期不因析构而阻塞。

---

### **3. 实现纯异步任务**
若需任务完全不阻塞当前线程，可手动管理线程：
#### **方法一：`std::packaged_task` + `std::thread`**
  ```cpp
  template<typename Func, typename... Args>
  auto AsyncTask(Func&& func, Args&&... args) -> std::future<decltype(func(args...))> {
      using RetType = decltype(func(args...));
      std::packaged_task<RetType()> task(std::bind(func, args...));
      auto future = task.get_future();
      std::thread(std::move(task)).detach(); // 分离线程
      return future;
  }

  void Test() {
      auto future = AsyncTask([]{
          std::this_thread::sleep_for(3s);
          std::cout << "Async task done" << std::endl;
      });
      // future析构时不会阻塞，但需确保主线程存活足够久
  }
  ```
  **注意**：分离线程需确保主线程不会提前退出，否则任务可能未完成。
  如果想实现一个纯异步操作，可以将打包好的packaged_task在绑定future后，再启动线程。
##### 紧急插入我的理解
  
  我现在已经隐约明白了，packaged_task是将可执行操作打包成一个任务（本身也是可调用对象，只是拥有了更多功能），但是任务只是一件事，不代表我已经启动它了。因而可以在启动之前，与future绑定，再由thread启动任务。thread是线程，能直接启动任何可调用对象。
  
  async是异步操作，与thread不同，它可以异步执行任务并直接返回future对象，即既能启动又能绑定。但是async却有着自己的缺陷（这一部分不太了解）。
  
  bind则是将函数打包成统一的可调用对象，经常结合模板、完美转发与万能引用一起使用。
```cpp
        auto task = std::make_shared<std::packaged_task<ReturnType()>>
        (
            // std::forward返回f与args的完美转发 形态的f和args
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );
```
  
  promise仍然有些迷惑。我目前的理解是，future与packaged_task结合，是future与一个函数返回值绑定，本质来说是future与函数操作绑定在一起；future与promise结合则是只和promise这一个值结合，换句话说，future只与一个变量绑定，大幅提高了灵活性。
```cpp
  #include <iostream>
#include <thread>
#include <future>

void calculate(std::promise<int> result_promise) {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    result_promise.set_value(42); // 手动设置结果
}

int main() {
    // 1. 创建一个 promise 对象
    std::promise<int> result_promise;

    // 2. 获取与 promise 关联的 future
    std::future<int> result = result_promise.get_future();

    // 3. 将 promise 移动到线程中
    std::thread t(calculate, std::move(result_promise));
    t.detach(); // 或 t.join()

    // 4. 通过 future 获取结果（会阻塞直到任务完成）
    std::cout << "Result: " << result.get() << std::endl;

    return 0;
}
```

##### 紧急评判我的理解（AI）
您的理解大部分是正确的，我来帮您进一步梳理和补充关键细节：

1. **`packaged_task` 的深层机制**
- 本质是一个包装器+存储容器，不仅打包任务，还会自动创建关联的 `future` 对象
- 当任务执行完成后，返回值或异常会被自动存储到共享状态中
- 典型应用场景：需要将任务与线程池调度解耦时使用
- 示例中 `std::bind` 的使用需要注意参数生命周期问题，现代 C++ 更推荐用 lambda 捕获：
```cpp
auto task = std::make_shared<std::packaged_task<ReturnType()>>(
    [f = std::forward<F>(f), ...args = std::forward<Args>(args)]() {
        return f(args...);
    }
);
```

2. **`promise` 的核心价值**
- 手动控制的结果传递通道，与 `future` 形成「生产者-消费者」模型
- 相比 `packaged_task` 的自动关联，`promise` 提供更细粒度的控制：
```cpp
std::promise<int> p;
auto f = p.get_future();

// 生产者线程
std::thread([&p]{
    try {
        p.set_value(42);  // 也可 set_exception
    } catch(...) {
        p.set_exception(std::current_exception());
    }
}).detach();

// 消费者
f.wait();
std::cout << f.get(); // 42
```
- 适用场景：需要跨多个线程设置结果、处理复杂同步逻辑、实现自定义通信协议

3. **`async` 的潜在缺陷**
- 启动策略的不确定性：默认 `std::launch::async | std::launch::deferred` 可能导致延迟执行
- 线程资源耗尽时可能阻塞（取决于实现）
- 异常传播机制的特殊性：在 `future` 析构时可能隐式等待
- 生命周期管理陷阱：返回的 `future` 持有共享状态，最后一个引用离开时才释放资源

4. **各组件对比表**

| 特性                | thread         | async          | packaged_task  | promise       |
|---------------------|----------------|----------------|----------------|---------------|
| 直接控制线程        | ✔️             | ❌             | ❌             | ❌            |
| 自动关联 future     | ❌             | ✔️             | ✔️             | ❌            |
| 手动设置结果        | ❌             | ❌             | ❌             | ✔️            |
| 异常自动捕获        | ❌             | ✔️             | ✔️             | ❌            |
| 可移动性            | ❌             | ✔️             | ✔️             | ✔️            |
| 典型应用场景        | 原始线程控制   | 快速异步任务   | 任务队列       | 复杂同步逻辑  |

5. **现代 C++ 并发编程建议**
- 优先使用 `async` 实现简单异步，但要注意返回的 `future` 必须被持有
- 需要精细控制时使用 `packaged_task` + `thread` 组合
- 当遇到以下情况时使用 `promise`：
  - 结果需要跨多个线程设置
  - 需要实现非函数式的异步操作（如事件驱动）
  - 要处理复杂的异常传播路径
- 使用 `std::bind` 时注意完美转发的正确写法：
```cpp
auto binder = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
// 更安全的现代写法：
auto task = [f = std::forward<F>(f), ...args = std::forward<Args>(args)] { 
    return f(args...); 
};
```

6. **生命周期注意事项**
- `packaged_task` 必须存活到任务执行完成
- `promise` 的生存期必须长于设置结果的线程
- `async` 返回的 `future` 析构时会隐式等待，可能造成意外阻塞

通过理解这些机制的内在联系，您可以根据具体场景选择最合适的工具：需要「即发即忘」用 `async`，需要任务队列用 `packaged_task`，需要精细控制用 `promise`，三者最终都通过 `future` 实现结果的异步获取。
  

---

### **4. 合理管理`future`生命周期**
- **传递`future`引用**：延长`future`生命周期，避免过早析构。
  ```cpp
  void func1(std::future<int>& ref) {
      ref = std::async(std::launch::async, []{ return 42; });
  }

  void func2(std::future<int>& ref) {
      int result = ref.get(); // 需要时阻塞获取结果
  }
  ```

---

### **5. 最佳实践**
1. **明确策略**：使用`std::launch::async`确保任务异步执行，而非默认策略（可能延迟执行）。
2. **避免阻塞析构**：若需任务后台运行，不关心结果，使用分离线程或全局`future`容器。
3. **锁的顺序**：避免主线程与异步任务竞争同一锁，尤其注意`future`析构点。
4. **结果收集**：在需要任务结果时调用`future::get()`，而非依赖析构隐式等待。

---

### **总结**
C++的`std::async`并非“纯异步”，其行为受`std::future`生命周期影响。理解共享状态和析构机制是避免阻塞和死锁的关键。在需要精细控制时，可结合`std::thread`和`std::packaged_task`实现更灵活的异步逻辑，同时注意资源管理和线程安全。