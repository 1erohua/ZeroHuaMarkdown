# 三剑客之间的关系

在 C++ 并发编程中，`std::future`、`std::promise` 和 `std::async` 是三个密切关联的核心组件，它们共同构成了异步操作的框架。以下是详细解释和它们之间的关系：

---

### 一、核心组件详解

#### 1. `std::future<T>`
- **作用**：表示一个异步操作的未来结果，提供访问异步操作结果的机制。
- **关键特性**：
  - 通过 `get()` 方法阻塞等待结果（只能调用一次）。
  - 通过 `valid()` 检查是否关联有效共享状态。
  - 通过 `wait()`/`wait_for()`/`wait_until()` 进行等待操作。
- **典型用途**：获取由 `std::async`、`std::packaged_task` 或 `std::promise` 设置的异步结果。

```cpp
std::future<int> fut = std::async([]{ return 42; });
int result = fut.get(); // 阻塞直到结果就绪
```

---

#### 2. `std::promise<T>`
- **作用**：用于手动设置异步操作的结果（值或异常），与 `future` 配对使用。
- **关键方法**：
  - `set_value()`：设置结果值。
  - `set_exception()`：设置异常。
  - `get_future()`：获取关联的 `future` 对象。
- **典型用途**：在自定义线程逻辑中显式传递结果。

```cpp
std::promise<int> prom;
std::future<int> fut = prom.get_future();

// 以std::promise<int>为参数的函数，作为线程进行调用
std::thread t([&prom]{ 
    prom.set_value(42); 
});
// 可以认为是在两个不同线程之间进行通信
// 由B线程发起的异步A线程
// 可以在A线程，通过if——else，根据不同的条件设置不同的prom值，从而指出A线程的执行情况
// 再由B线程，通过future获取prom值的结果，从而得到A线程的执行情况
// 因而prom的泛型往往是一个结构体或者类，这样才能有更多的信息可以传递

t.detach();

int result = fut.get(); // 获取结果
```

---

#### 3. `std::async`
- **作用**：异步启动任务，返回关联的 `future` 对象。
- **启动策略**：
  - `std::launch::async`：强制在新线程执行。
  - `std::launch::deferred`：延迟执行（调用 `future::get()` 时运行）。
  - 默认策略：由实现决定（通常是 `async | deferred`）。
- **典型用途**：快速封装异步任务，无需手动管理线程。

```cpp
auto fut = std::async(std::launch::async, []{
    return std::sqrt(25.0);
});
double result = fut.get(); // 5.0
```

---

### 二、三者关系

1. **协作流程**：
   - **生产者**：通过 `std::promise::set_value()` 或 `std::async` 生成结果。
   - **消费者**：通过 `std::future::get()` 获取结果。
   - **中间桥梁**：共享状态（Shared State）存储结果和同步信息。

   ```
   async/promise → 设置结果 → 共享状态 → future → 获取结果
   ```

2. **依赖关系**：
   - `std::async` 内部可能使用 `std::promise` 和 `std::future` 的组合。
   - `std::promise` 必须通过 `get_future()` 与 `future` 绑定。

---

### 三、核心机制：共享状态（Shared State）
- **作用**：
  - 存储异步操作的结果（值或异常）。
  - 管理线程同步（如 `future::get()` 的阻塞）。
- **生命周期**：
  - 由最后一个关联的 `future` 或 `promise` 释放时销毁。
  - 确保结果在需要时可用。

---

### 四、异常处理
- **通过 `future` 传播异常**：
  - 异步任务中未捕获的异常会通过 `future::get()` 重新抛出。
  - 可通过 `promise::set_exception()` 手动设置异常。

```cpp
std::promise<void> prom;
auto fut = prom.get_future();
std::thread([&prom]{
    try {
        // 可能抛出异常的操作
        prom.set_value();
    } catch (...) {
        prom.set_exception(std::current_exception());
    }
}).detach();

try {
    fut.get();
} catch (const std::exception& e) {
    // 处理异常
}
```

---

### 五、使用场景对比

| 组件          | 适用场景                                                                 |
|---------------|-------------------------------------------------------------------------|
| `std::async`  | 快速启动简单异步任务，无需手动管理线程和结果传递。                       |
| `std::promise`| 需要显式控制结果设置的场景（如自定义线程池或复杂同步逻辑）。             |
| `std::future` | 所有需要获取异步结果的场景（与 `async` 或 `promise` 配合使用）。         |

---

### 六、注意事项
1. **`future::get()` 的独占性**：
   - `get()` 只能调用一次，调用后 `future` 变为无效。
2. **线程安全**：
   - `promise` 和 `future` 的非 const 方法非线程安全。
3. **资源管理**：
   - 确保 `promise` 在设置结果前不被销毁。
4. **死锁风险**：
   - 避免在未分离或未连接的线程中未设置 `promise` 结果。

---

### 七、完整示例
```cpp
#include <iostream>
#include <future>
#include <thread>

// 使用 async 和 future
void async_example() {
    auto fut = std::async(std::launch::async, []{
        return "Hello from async!";
    });
    std::cout << fut.get() << std::endl;
}

// 使用 promise 和 future
void promise_example() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread t([&prom]{
        std::this_thread::sleep_for(std::chrono::seconds(1));
        prom.set_value(42);
    });

    std::cout << "Waiting for result..." << std::endl;
    std::cout << "Result: " << fut.get() << std::endl;
    t.join();
}

int main() {
    async_example();
    promise_example();
    return 0;
}
```

---

### 总结
- **`std::future`** 是异步结果的消费者接口。
- **`std::promise`** 是异步结果的生产者接口。
- **`std::async`** 是简化异步任务创建的高级工具。
- 三者通过共享状态协同工作，实现线程间安全的数据传递和同步。

# 将任务与future关联
在 C++ 并发编程中，`std::future` 和 `std::packaged_task` 是密切相关的组件，它们共同用于封装异步任务并管理其结果。以下是它们的详细关系及使用场景分析。

---

### 一、`std::packaged_task` 的核心作用
`std::packaged_task` 是一个类模板，用于**将任意可调用对象（函数、Lambda、函数对象等）包装成一个异步任务**，并将其执行结果与一个 `std::future` 对象绑定。它的核心功能是为任务提供结果传递的桥梁。

#### 关键特性：
1. **包装可调用对象**：
   - 接受一个函数签名（如 `int(int, int)`）作为模板参数，并包装对应的可调用对象。
2. **关联 `std::future`**：
   - 通过 `get_future()` 方法获取与任务关联的 `std::future`，用于后续获取结果。
3. **自动管理共享状态**：
   - 当任务执行时，结果（或异常）会自动存储到共享状态中，无需手动设置（如 `std::promise`）。

---

### 二、`std::future` 与 `std::packaged_task` 的关系

#### 1. **协作流程**
- **步骤 1**：创建 `std::packaged_task`，包装一个任务（如函数或 Lambda）。
- **步骤 2**：通过 `get_future()` 获取关联的 `std::future`。
- **步骤 3**：将 `packaged_task` 传递给线程或其他执行上下文（如线程池）。
- **步骤 4**：执行 `packaged_task`，结果自动存入共享状态。
- **步骤 5**：通过 `future.get()` 获取结果。

```cpp
#include <iostream>
#include <future>
#include <thread>

int add(int a, int b) {
    return a + b;
}

int main() {
    // 步骤 1：创建 packaged_task，包装任务 add
    std::packaged_task<int(int, int)> task(add);

    // 步骤 2：获取关联的 future
    std::future<int> fut = task.get_future();

    // 步骤 3：将任务移动到线程中执行
    std::thread t(std::move(task), 2, 3);
    t.join();

    // 步骤 5：获取结果
    std::cout << "Result: " << fut.get() << std::endl; // 输出 5
    return 0;
}
```

#### 2. **共享状态绑定**
- `std::packaged_task` 和 `std::future` 共享同一个**共享状态（Shared State）**。
- 当 `packaged_task` 被调用时，其结果或异常会自动存储到共享状态中。
- `future` 通过共享状态获取结果，并处理同步（如阻塞等待）。

---

### 三、`std::packaged_task` 与 `std::promise` 的对比
虽然两者都可以生成 `std::future`，但设计目标不同：

| 特性                | `std::packaged_task`                          | `std::promise`                          |
|---------------------|-----------------------------------------------|------------------------------------------|
| **用途**            | 封装可调用对象，自动传递结果                  | 手动设置结果（值或异常）                 |
| **结果设置方式**    | 自动（通过执行任务）                          | 手动（`set_value()` 或 `set_exception()`） |
| **灵活性**          | 适用于需要直接执行任务的场景                  | 适用于需要显式控制结果生成的场景         |
| **典型场景**        | 线程池任务调度、延迟执行                      | 跨线程手动传递结果                       |

---

### 四、`std::packaged_task` 的核心操作

#### 1. 创建与绑定任务
```cpp
// 包装一个 Lambda
std::packaged_task<int()> task([]{
    return 42;
});

// 包装一个函数对象
struct Multiply {
    int operator()(int a, int b) { return a * b; }
};
std::packaged_task<int(int, int)> task(Multiply{});
```

#### 2. 执行任务
- `packaged_task` 本身是一个可调用对象，通过 `operator()` 执行。
- 执行后，结果存入共享状态，`future` 可获取。

```cpp
std::packaged_task<int(int, int)> task([](int a, int b) { return a * b; });
std::future<int> fut = task.get_future();

// 执行任务（可以直接调用，也可以传递给线程）
task(2, 3); // 手动执行
// 或
std::thread t(std::move(task), 2, 3);
t.join();

std::cout << fut.get() << std::endl; // 输出 6
```

#### 3. 移动语义
- `std::packaged_task` 不可复制，但支持移动语义。
- 任务所有权可以转移到线程或其他容器（如任务队列）。

```cpp
std::packaged_task<void()> task1([]{ /* ... */ });
std::packaged_task<void()> task2 = std::move(task1); // 合法
```

---

### 五、`std::packaged_task` 的典型使用场景

#### 1. 线程池任务调度
将任务封装为 `packaged_task`，提交到线程池队列，并通过关联的 `future` 异步获取结果。

```cpp
// 示例：简单线程池任务提交
std::vector<std::packaged_task<int()>> tasks;
std::vector<std::future<int>> futures;

// 提交任务
tasks.emplace_back([]{ return 42; });
futures.push_back(tasks.back().get_future());

// 线程池取出任务并执行
std::thread worker([&tasks]{
    for (auto& task : tasks) {
        task();
    }
});
worker.join();

// 获取结果
std::cout << futures[0].get() << std::endl; // 42
```

#### 2. 延迟执行任务
将任务存储起来，稍后在特定条件下执行。

```cpp
std::packaged_task<int()> task([]{ return 42; });
std::future<int> fut = task.get_future();

// 稍后执行
std::thread t(std::move(task));
t.detach();

// 在需要结果时阻塞等待
if (fut.valid()) {
    std::cout << fut.get() << std::endl;
}
```

#### 3. 组合多个异步任务
通过多个 `future` 和 `packaged_task` 实现复杂依赖关系。

---

### 六、异常处理
- 如果 `packaged_task` 的执行过程中抛出异常，异常会被捕获并存储到共享状态。
- 通过 `future.get()` 获取结果时，异常会重新抛出。

```cpp
std::packaged_task<void()> task([]{
    throw std::runtime_error("Something went wrong!");
});

std::future<void> fut = task.get_future();
std::thread t(std::move(task));
t.join();

try {
    fut.get();
} catch (const std::exception& e) {
    std::cerr << "Caught exception: " << e.what() << std::endl;
}
```

---

### 七、`std::packaged_task` 与 `std::async` 的区别

| 特性                  | `std::packaged_task`                          | `std::async`                                  |
|-----------------------|-----------------------------------------------|-----------------------------------------------|
| **任务执行控制**      | 手动控制（需显式调用或传递给线程）            | 自动执行（依赖启动策略）                      |
| **灵活性**            | 高（可自由管理任务执行时机）                  | 低（由 `async` 管理执行）                     |
| **适用场景**          | 需要任务对象传递或复杂调度逻辑                | 简单异步任务（无需手动线程管理）              |
| **资源管理**          | 需手动管理线程或任务队列                      | 自动管理线程（取决于启动策略）                |
| **结果获取**          | 通过关联的 `future`                           | 直接返回 `future`                             |

---

### 八、总结
- **`std::packaged_task`**：
  - 是一个任务包装器，将可调用对象与 `future` 绑定。
  - 提供对任务执行的精细控制（如延迟执行、线程池调度）。
- **`std::future`**：
  - 是结果的消费者，用于从 `packaged_task` 中获取异步结果。
- **关系本质**：
  - `packaged_task` 是生产者（生成结果），`future` 是消费者（获取结果）。
  - 两者通过共享状态同步，实现线程安全的异步结果传递。

---

### 完整示例：结合线程池
```cpp
#include <iostream>
#include <future>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>

class ThreadPool {
public:
    ThreadPool(size_t num_threads) : stop(false) {
        for (size_t i = 0; i < num_threads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::packaged_task<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(queue_mutex);
                        condition.wait(lock, [this] {
                            return stop || !tasks.empty();
                        });
                        if (stop && tasks.empty()) return;
                        task = std::move(tasks.front());
                        tasks.pop();
                    }
                    task(); // 执行任务
                }
            });
        }
    }

    template <typename F>
    std::future<void> enqueue(F&& f) {
        std::packaged_task<void()> task(std::forward<F>(f));
        std::future<void> fut = task.get_future();
        {
            std::lock_guard<std::mutex> lock(queue_mutex);
            tasks.emplace(std::move(task));
        }
        condition.notify_one();
        return fut;
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for (std::thread& worker : workers) {
            worker.join();
        }
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::packaged_task<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

int main() {
    ThreadPool pool(4);

    auto fut = pool.enqueue([]{
        std::cout << "Task executed in thread pool" << std::endl;
    });

    fut.wait(); // 等待任务完成
    return 0;
}
```

---

### 关键点总结
- `std::packaged_task` 是连接任务和 `std::future` 的桥梁，提供对异步任务的高级抽象。
- 与 `std::promise` 不同，`packaged_task` 自动管理结果传递，无需手动调用 `set_value()`。
- 在线程池、任务队列或需要延迟执行的场景中，`packaged_task` 比 `std::async` 更灵活。

# std::shard_future
在 C++ 并发编程中，`std::shared_future` 是 `std::future` 的可拷贝版本，允许多个消费者共享同一个异步操作的结果。它解决了 `std::future` 只能被单次消费的限制，适用于需要多个线程或组件访问同一异步结果的场景。

---

### 一、`std::shared_future` 的核心特性

#### 1. **与 `std::future` 的区别**
| 特性               | `std::future<T>`                     | `std::shared_future<T>`                |
|--------------------|--------------------------------------|-----------------------------------------|
| **拷贝语义**       | 不可拷贝，仅支持移动（Move-only）    | 可拷贝（Copyable）                      |
| **结果访问次数**   | `get()` 只能调用一次                 | `get()` 可多次调用                      |
| **多线程共享**     | 不允许多线程直接共享                 | 允许多线程安全共享                      |
| **共享状态所有权** | 独占共享状态（转移所有权）           | 共享共享状态（多实例共享同一状态）      |

#### 2. **核心功能**
- 允许多个消费者从同一共享状态获取结果。
- 线程安全：多个线程可以同时调用 `get()` 获取结果。
- 支持延迟结果传递（与 `std::promise` 或 `std::packaged_task` 配合使用）。

---

### 二、`std::shared_future` 的创建方式

#### 1. 从 `std::future` 转换
通过 `std::future::share()` 方法生成 `std::shared_future`，原始 `future` 将失效（所有权转移）。

```cpp
std::future<int> fut = std::async([]{ return 42; });
std::shared_future<int> shared_fut = fut.share(); // fut 变为 invalid

// 多线程共享 shared_fut
std::thread t1([shared_fut]{ std::cout << shared_fut.get(); });
std::thread t2([shared_fut]{ std::cout << shared_fut.get(); });
t1.join(); t2.join(); // 输出 42 42
```

#### 2. 直接构造
从已有的 `std::shared_future` 拷贝构造或从 `std::promise` 获取。

```cpp
std::promise<int> prom;
std::shared_future<int> shared_fut = prom.get_future().share();

std::thread([&prom]{ prom.set_value(100); }).detach();

// 共享到多个线程
std::thread([shared_fut]{ std::cout << shared_fut.get(); }).join();
std::thread([shared_fut]{ std::cout << shared_fut.get(); }).join();
```

---

### 三、典型使用场景

#### 1. **多线程共享同一结果**
多个线程需要等待并处理同一个异步操作的结果（如全局配置加载、分布式计算）。

```cpp
std::shared_future<std::string> config_future = load_config_async();

// 多个组件线程等待配置
std::thread ui_thread([config_future]{
    auto config = config_future.get();
    update_ui(config);
});

std::thread network_thread([config_future]{
    auto config = config_future.get();
    setup_network(config);
});

ui_thread.join();
network_thread.join();
```

#### 2. **多次访问结果**
需要多次访问同一结果（如历史计算结果缓存）。

```cpp
std::shared_future<double> result_future = std::async(calculate_pi, 1000);

// 多次使用结果
double pi1 = result_future.get();
double pi2 = result_future.get(); // 合法，不会失效
```

#### 3. **结合条件变量实现通知机制**
通过 `shared_future` 实现一次性事件的广播通知。

```cpp
std::promise<void> start_promise;
std::shared_future<void> start_future = start_promise.get_future().share();

// 多个线程等待启动信号
std::thread worker1([start_future]{
    start_future.wait(); // 阻塞直到 set_value()
    do_work();
});

std::thread worker2([start_future]{
    start_future.wait();
    do_work();
});

// 触发所有线程启动
start_promise.set_value();
worker1.join(); worker2.join();
```

---

### 四、线程安全性
- **`get()` 的线程安全**：多个线程同时调用 `shared_future::get()` 是安全的。
- **写操作的线程安全**：`shared_future` 本身不可修改，共享状态由生产者（如 `promise`）设置，需确保生产者线程安全。

---

### 五、异常处理
- 与 `std::future` 类似，异步任务中的异常会通过 `shared_future::get()` 传播。
- 所有共享的 `shared_future` 实例在调用 `get()` 时都可能抛出异常。

```cpp
std::shared_future<void> err_future = std::async([]{
    throw std::runtime_error("Oops!");
}).share();

std::thread([err_future]{
    try {
        err_future.get();
    } catch (const std::exception& e) {
        std::cerr << "Thread 1: " << e.what() << std::endl;
    }
});

std::thread([err_future]{
    try {
        err_future.get();
    } catch (const std::exception& e) {
        std::cerr << "Thread 2: " << e.what() << std::endl;
    }
}).join();
```

---

### 六、性能与注意事项
1. **共享状态开销**：
   - `shared_future` 的共享状态需要额外的同步和引用计数管理，可能带来轻微性能开销。
2. **生命周期管理**：
   - 确保所有 `shared_future` 实例销毁前，共享状态未被提前释放。
3. **避免悬空引用**：
   - 如果共享状态由临时对象生成（如 `std::async` 返回的 `future`），需确保其生命周期足够长。

---

### 七、完整示例：多线程共享计算结果

```cpp
#include <iostream>
#include <future>
#include <vector>
#include <thread>

int main() {
    // 异步计算 1+2+...+100
    std::future<int> fut = std::async([]{
        int sum = 0;
        for (int i = 1; i <= 100; ++i) sum += i;
        return sum;
    });

    // 转换为 shared_future
    std::shared_future<int> shared_fut = fut.share();

    // 创建 5 个线程，每个线程读取结果
    std::vector<std::thread> threads;
    for (int i = 0; i < 5; ++i) {
        threads.emplace_back([shared_fut, i]{
            int result = shared_fut.get();
            std::cout << "Thread " << i << ": Sum = " << result << std::endl;
        });
    }

    for (auto& t : threads) t.join();
    return 0;
}
```

**输出**：
```
Thread 2: Sum = 5050
Thread 1: Sum = 5050
Thread 0: Sum = 5050
Thread 3: Sum = 5050
Thread 4: Sum = 5050
```

---

### 总结
- **`std::shared_future`** 是 `std::future` 的共享版本，支持多消费者模型。
- **适用场景**：多线程共享结果、多次访问结果、广播通知。
- **核心优势**：线程安全、可拷贝、结果可重复访问。
- **实现本质**：通过引用计数管理共享状态，确保生命周期安全。
  >但是大家要注意，如果一个`future`被移动给两个`shared_future`是错误的。
  


# 公司级线程池？

1. **减少线程创建/销毁的开销**：复用固定数量的线程处理多个任务。
    
2. **任务与线程解耦**：通过队列管理任务，线程按需获取任务执行。
    
3. **动态扩展能力**：根据任务负载动态调整线程数量（可选）。
    
4. **异步结果传递**：通过 `std::future` 或 `std::shared_future` 获取任务结果。

下面的代码有太多能学习的东西了，无论是泛型模板，还是完美转发，亦或是智能指针，还是线程调度

```cpp
#include <queue>
#include <vector>
#include <thread>
#include <future>

class ThreadPool{
private:
    std::vector<std::thread> _pool; // 线程池
    std::queue<std::packaged_task<void()>> _tasks; // 任务队列————从任务队列取出任务执行时不关心返回值，因而全设为空
    std::mutex mtx; // 互斥锁，当访问_tasks与_pool时，需要给它们加锁

    std::atomic_bool _running; // 线程池还在跑吗
    std::atomic<unsigned int> _AvailableThreadCount; // 可调用线程数 

    // 条件变量
    std::condition_variable cv;

    // 删除拷贝构造赋值
    ThreadPool(const ThreadPool&) = delete;
    ThreadPool& operator=(const ThreadPool&) = delete;

    // 私有函数, 根据安全性，构造函数、线程池启动、线程池停止必须为私有
    ThreadPool(unsigned int _use_count = 5):_running(true){
        if(_use_count < 1) {
            _AvailableThreadCount.store(1); 
        }
        else if(_use_count > std::thread::hardware_concurrency()){
            _AvailableThreadCount.store(std::thread::hardware_concurrency());
        }
        else _AvailableThreadCount.store(_use_count);
        start();
    }

    void start(){
        // 按照可用线程数量，塞进线程池
        for(unsigned int i=0; i<_AvailableThreadCount.load(); i++){
            // 注意，_pool是std::vector<std::thread> 类型，使用emplace_back塞入
            // 会直接在容器内进行构造
            // 而Thread是一旦构造就会直接异步运行的
            _pool.emplace_back([this](){
                // 以下都是函数内容, 即完全可以写在另一个函数的
                while(this->_running.load()){

                std::packaged_task<void()> task;

                    {
                        std::unique_lock<std::mutex> getTask(this->mtx);
                        this->cv.wait(getTask, [this](){
                            // 唤醒条件：
                            // 队列不为空，或线程池停止运行
                            // 线程池停止运行时，必须执行最后一波
                            return !(this->_running.load()) || !this->_tasks.empty();  
                        });

                        // 唤醒后这里已经加锁
                        // 再进行等待队列是否为空判断，如果为空，那么代表线程池已经停止运行了，此时退出即可
                        // 只要能到这里，要么等待队列非空，要么线程池停运
                        if(this->_tasks.empty()){
                            return;
                        }

                        // 到达这里，说明等待队列非空，那么进行从队列取任务
                        task = std::move(this->_tasks.front()); // 拿出第一个任务
	                        this->_tasks.pop();
                    }// 锁释放
                
                this->_AvailableThreadCount.fetch_sub(1); // 调用任务前，先减少线程数量
                task();
                this->_AvailableThreadCount.fetch_add(1); // 调用任务后，回调可用线程数

                }
            });
        }
    }

    void stop(){
        _running.store(false);

        cv.notify_all(); // 唤醒所有线程

        // 阻塞等待所有线程完成
        for(auto& th:_pool){
            if(th.joinable()){
                th.join();
            }
        }
    }

public:
    // 单例构造————是在内部构造！不是给构造函数加上static
    static ThreadPool& GetThreadPool(){
        static ThreadPool thread_pool;
        return thread_pool;
    }

    ~ThreadPool(){
        stop();
    }

    // 这一段要多练练
    // 提交任务到等待队列
    template<typename F, typename ...Args>
    auto commit(F&& f, Args&&... args)->std::future<decltype(f(args...))>{
        using ReturnType = decltype(f(args...));

        // 如果线程池不在工作
        if(!_running.load()){
            return std::future<ReturnType>{};
        }

        // 包装成任务(这里我们使用智能指针管理)、绑定future、加入队列、返回值
        // 这里必须使用智能指针, package_task不可拷贝；即便使用引用，离开本函数就会被销毁，后面执行多线程时就无法找到它
        // 使用智能指针还能保证它的生命周期
        auto task = std::make_shared<std::packaged_task<ReturnType()>>
        (
            // std::forward返回f与args的完美转发 形态的f和args
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<ReturnType> fut = task->get_future(); // 绑定future

        {
            // 加锁放进队列
            std::lock_guard<std::mutex> lock(mtx);
            this->_tasks.emplace([task](){
                // 这里不是直接将（*task）塞进队列，而是使用lambda，在lambda内进行调用，这样就直接抹去了返回值
                // 至于它的返回值，已经将由本函数的返回内容获取
                (*task)();
            })
        }
        cv.notify_one();

        return fut;
    }

    // 获取当前可用线程数
    int GetAvaliableThread(){
        return _AvailableThreadCount.load();
    }

	// 
	int GetTaskNum(){
		return _tasks.size();
	}

};

```

上面的线程池还是有缺陷的，比如当主线程结束，线程池进入销毁时，任务队列还有很多任务时，是没办法全部执行完成的。