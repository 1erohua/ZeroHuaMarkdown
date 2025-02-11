# C++ 多线程中的 Actor 设计模式详解

Actor 模式是一种**基于消息传递的并发模型**，其核心思想是**通过隔离状态和消息通信实现线程安全**。每个 Actor 是一个独立实体，拥有私有状态和消息队列，通过异步消息与其他 Actor 交互，天然避免了共享内存和锁竞争问题。

---

#### Actor 模式核心机制
1. **消息队列**：每个 Actor 拥有线程安全的队列，存储待处理消息。
2. **私有状态**：Actor 内部状态只能通过处理消息修改。
3. **消息处理循环**：Actor 线程持续从队列中取消息并处理。
4. **异步通信**：Actor 之间通过发送不可变消息交互，无直接方法调用。

---

### C++ 实现 Actor 模式的示例

#### 1. 定义 Actor 基类
```cpp
#include <iostream>
#include <thread>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <future>
#include <any>

// 消息基类（实际使用中可定义具体消息类型）
struct Message {
    virtual ~Message() = default;
};

class Actor {
public:
    Actor() : running_(true), thread_([this] { run(); }) {}
    virtual ~Actor() {
        send([this] { running_ = false; }); // 发送停止消息
        thread_.join();
    }

    template<typename F>
    void send(F&& message) {
        {
            std::lock_guard<std::mutex> lock(mutex_);
            messages_.push(std::forward<F>(message));
        }
        condition_.notify_one();
    }

protected:
    virtual void process_message(const std::any& msg) = 0;

private:
    std::queue<std::any> messages_;
    std::mutex mutex_;
    std::condition_variable condition_;
    bool running_;
    std::thread thread_;

    void run() {
        while (running_) {
            std::unique_lock<std::mutex> lock(mutex_);
            condition_.wait(lock, [this] { return !messages_.empty(); });

            auto msg = std::move(messages_.front());
            messages_.pop();
            lock.unlock();

            process_message(msg);
        }
    }
};
```

#### 2. 实现具体 Actor（以银行账户为例）
```cpp
class BankAccount : public Actor {
public:
    BankAccount() : balance_(0) {}

    // 查询余额（异步返回）
    std::future<int> get_balance() {
        auto promise = std::make_shared<std::promise<int>>();
        send([this, promise] {
            promise->set_value(balance_);
        });
        return promise->get_future();
    }

    // 存款操作
    void deposit(int amount) {
        send([this, amount] {
            balance_ += amount;
            std::cout << "Deposited " << amount 
                      << ", new balance: " << balance_ << std::endl;
        });
    }

    // 取款操作
    void withdraw(int amount) {
        send([this, amount] {
            if (balance_ >= amount) {
                balance_ -= amount;
                std::cout << "Withdrew " << amount 
                          << ", new balance: " << balance_ << std::endl;
            } else {
                std::cout << "Insufficient funds!" << std::endl;
            }
        });
    }

protected:
    void process_message(const std::any& msg) override {
        if (auto func = std::any_cast<std::function<void()>>(&msg)) {
            (*func)();
        }
    }

private:
    int balance_;
};
```

#### 3. 使用示例
```cpp
int main() {
    BankAccount account;

    // 异步操作
    account.deposit(100);
    account.withdraw(50);
    account.deposit(200);

    // 异步获取余额
    auto balance_future = account.get_balance();
    std::cout << "Current balance: " << balance_future.get() << std::endl;

    return 0;
}
```

---

### 关键设计要点
1. **线程安全队列**：使用 `mutex` + `condition_variable` 实现高效消息传递。
2. **类型擦除**：通过 `std::any` 存储不同类型的消息。
3. **异步返回**：结合 `std::promise`/`std::future` 实现跨线程结果返回。
4. **生命周期管理**：通过发送特殊消息优雅终止 Actor。

---

### Actor 模式适用场景
- **高并发服务**：如游戏服务器、聊天系统
- **状态隔离需求**：如金融交易系统
- **分布式系统**：本地 Actor 模型可无缝扩展为分布式系统

### 性能优化方向
- **批量处理**：合并多个消息一次性处理
- **无锁队列**：使用 `boost::lockfree::queue` 提升性能
- **线程池**：多个 Actor 共享线程池减少资源消耗

Actor 模式通过强制消息传递机制，将并发复杂度封装在消息通道中，是构建高可靠并发系统的有效范式。

您观察得非常准确！虽然在 Actor 模式的理论设计中强调**无共享状态**，但在具体实现时消息队列本身仍需线程安全机制。这是理论模型与工程实现的差异点，让我们深入探讨这个关键矛盾：

---

### 锁在 Actor 实现中的本质作用
1. **消息队列的临界区保护**  
   当多个线程同时向同一个 Actor 发送消息时，消息队列的`push`操作需要互斥锁保证原子性。这是**跨线程消息传递**的必然需求，与 Actor 模式本身无关。

2. **状态访问的天然串行化**  
   Actor 内部的消息处理循环（`run()`）通过单线程顺序执行消息，使得对`balance_`等私有状态的访问**天然线程安全**，此时无需额外锁。

3. **条件变量的唤醒机制**  
   `condition_variable`用于在没有消息时让线程休眠，避免忙等待消耗 CPU，属于性能优化手段而非核心模式要求。


# 那这样一来不是还是使用锁了？

### 无锁实现的可行性（以 C++ 为例）
#### 方案一：无锁队列替换
```cpp
#include <boost/lockfree/queue.hpp> 

class LockfreeActor {
    boost::lockfree::queue<std::function<void()>> queue_{1000}; // 无锁队列

    void send(std::function<void()> msg) {
        while (!queue_.push(msg)) {} // 自旋直到成功
        // 可配合无锁条件变量唤醒
    }
};
```
- **优势**：消除锁竞争，提升吞吐量
- **代价**：可能引入忙等待，增加 CPU 占用

#### 方案二：线程绑定优化
```cpp
class ThreadAffinityActor {
    std::atomic<bool> has_message_{false};
    
    void send(std::function<void()> msg) {
        queue_.push(msg); // 需要无锁队列
        has_message_.store(true, std::memory_order_release);
        
        if (!is_processing_.exchange(true)) { // CAS 操作
            // 只有首个触发者启动处理
            std::thread([this] { process(); }).detach();
        }
    }
};
```
- **特点**：通过原子操作和 CAS 实现轻量级同步
- **适用场景**：高频短消息处理

---

### 工程实践中的权衡决策
| 方案类型       | 吞吐量 | 延迟 | CPU 占用 | 实现复杂度 | 适用场景              |
|----------------|--------|------|----------|------------|-----------------------|
| 互斥锁+条件变量| 中     | 低   | 低       | 低         | 通用型                |
| 无锁队列       | 高     | 中   | 高       | 中         | 高频消息              |
| 线程绑定       | 极高   | 高   | 中       | 高         | 实时系统/低延迟需求   |

---

### 现代 C++ 的优化实践
#### 使用 MPSC 队列（多生产者单消费者）
```cpp
template<typename T>
class MpscQueue {
    struct Node {
        std::atomic<Node*> next;
        T data;
    };

    std::atomic<Node*> head_;
    Node* tail_; // 仅消费者线程访问

public:
    void push(T item) { // 多线程安全
        auto new_node = new Node{nullptr, std::move(item)};
        Node* old_head = head_.exchange(new_node, std::memory_order_acq_rel);
        old_head->next.store(new_node, std::memory_order_release);
    }

    bool pop(T& item) { // 单线程消费
        Node* first = tail_->next.load(std::memory_order_acquire);
        if (first) {
            item = std::move(first->data);
            tail_ = first;
            delete tail_;
            return true;
        }
        return false;
    }
};
```
- **特性**：针对多生产者场景优化的无锁结构
- **内存序**：通过`memory_order_acq_rel`保证跨线程可见性

---

### Actor 模式的锁使用本质
1. **架构级无锁**：业务逻辑无需关心锁，所有并发复杂性被封装在消息通道中
2. **实现级可控锁**：将锁的粒度缩小到消息传递机制本身，与业务状态解耦
3. **可替换性**：锁实现方案可根据场景替换为无锁结构，不影响上层业务逻辑

---

### 生产环境建议
1. **优先使用成熟库**：  
   - [CAF (C++ Actor Framework)](https://github.com/actor-framework/actor-framework)
   - [SObjectizer](https://github.com/Stiffstream/sobjectizer)
   这些库已实现高度优化的消息通道。

2. **性能关键系统**：  
   可结合硬件特性实现：
   ```cpp
   // 利用线程局部存储优化
   thread_local Actor* current_actor = nullptr;

   // 使用内存屏障替代锁
   void send(Message msg) {
       __atomic_store_n(&queue.tail, msg, __ATOMIC_RELEASE);
       asm volatile("" ::: "memory"); // 编译器屏障
   }
   ```

3. **分布式扩展**：  
   通过包装网络层，使本地 Actor 的消息队列透明扩展为跨进程通信：
   ```cpp
   class NetworkActorProxy : public Actor {
       void send(Message msg) override {
           // 序列化消息并通过 TCP/UDP 发送
           socket.send(serialize(msg));
       }
   };
   ```

---

### 结论
Actor 模式中锁的使用是**消息通道的实现细节**，而非模式本身的核心特征。通过选择不同的底层同步机制，开发者可以在线程安全与性能之间找到最佳平衡点，这正是该模式强大适应性的体现。正如汽车发动机需要火花塞点火，但这不妨碍我们将其视为"内燃机"——关键在架构层面的设计哲学，而非具体组件的实现方式。


# 自写Actor邮件收发

```cpp
#include <string>
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <future>
#include <thread>
#include <chrono>

class Actor;

class Message {
private:
    std::string _message;
    std::promise<bool> isRead;

public:
    Message(std::string message) : _message(message) {}

    std::string ReadMessage() {
        isRead.set_value(true);
        return _message;
    }

    // 公有方法返回 shared_future
    std::shared_future<bool> getReadFuture() {
        return isRead.get_future().share();
    }
};

class Actor {
private:
    std::mutex mtx;
    std::condition_variable cv;
    std::queue<std::shared_ptr<Message>> email;

    bool running;
    std::thread _thread;

    void run() {
        while (running) {
            std::shared_ptr<Message> mes_ptr;
            {
                std::unique_lock<std::mutex> lock(mtx);
                cv.wait(lock, [this] {
                    return !this->email.empty() || !running;
                });

                if (email.empty()) break;

                mes_ptr = email.front();
                email.pop();
            }

            std::cout << mes_ptr->ReadMessage() << std::endl;
        }
    }

public:
    Actor() : running(true), _thread([this] { run(); }) {}

    ~Actor() {
        running = false;
        cv.notify_one();
        if (_thread.joinable()) {
            _thread.join();
        }
    }

    std::shared_future<bool> sendMessage(std::string message) {
        std::shared_ptr<Message> mes_ptr = std::make_shared<Message>(message);
        {
            std::lock_guard<std::mutex> lock(mtx);
            email.emplace(mes_ptr);
        }
        cv.notify_one();
        return mes_ptr->getReadFuture(); // 使用公有方法获取 future
    }
};

int main() {
    Actor actor;
    auto fut = actor.sendMessage("Hello, Actor!");
    fut.wait();
    if (fut.get()) {
        std::cout << "Message was read." << std::endl;
    }
    return 0;
}

```



# CSP模式
**通过通信来共享内存，而不是通过共享内存来通信**。

CSP和Actor类似，只不过CSP将消息投递给channel，至于谁从channel中取数据，发送的一方是不关注的。简单的说Actor在发送消息前是直到接收方是谁，而接受方收到消息后也知道发送方是谁，更像是邮件的通信模式。而csp是完全解耦合的。


下面只介绍简单的作为学习


```cpp
#include <iostream>
#include <queue>
#include <mutex>
#include <condition_variable>

template <typename T>
class Channel {
private:
    std::queue<T> queue_;
    std::mutex mtx_;
    std::condition_variable cv_producer_;
    std::condition_variable cv_consumer_;
    size_t capacity_;
    bool closed_ = false; // 默认通道关闭

public:
    Channel(size_t capacity = 0) : capacity_(capacity) {}
    ~Channel(){close();}

    bool send(T value) {
        std::unique_lock<std::mutex> lock(mtx_);
        cv_producer_.wait(lock, [this]() {
            // 对于无缓冲的channel，我们应该等待直到有消费者准备好
            return (capacity_ == 0 && queue_.empty()) || queue_.size() < capacity_ || closed_;
            });

        if (closed_) {
            return false;
        }

        queue_.push(value);
        cv_consumer_.notify_one();
        return true;
    }

    bool receive(T& value) {
        std::unique_lock<std::mutex> lock(mtx_);
        cv_consumer_.wait(lock, [this]() { return !queue_.empty() || closed_; });

        if (closed_ && queue_.empty()) {
            return false;
        }

        value = queue_.front();
        queue_.pop();
        cv_producer_.notify_one();
        return true;
    }

    void close() {
        std::unique_lock<std::mutex> lock(mtx_);
        closed_ = true;
        cv_producer_.notify_all();
        cv_consumer_.notify_all();
    }
};

// 示例使用
int main() {
    Channel<int> ch(10);  // 10缓冲的channel

    std::thread producer([&]() {
        for (int i = 0; i < 5; ++i) {
            ch.send(i);
            std::cout << "Sent: " << i << std::endl;
        }
        ch.close();
        });

    std::thread consumer([&]() {
        std::this_thread::sleep_for(std::chrono::milliseconds(500)); // 故意延迟消费者开始消费
        int val;
        while (ch.receive(val)) {
            std::cout << "Received: " << val << std::endl;
        }
        });

    producer.join();
    consumer.join();
    return 0;
}

```
个人看法，从实现来看，不在乎有哪些人，只在乎通道本身，并且由通道来的收发方法来划分生产者消费者；实际上任何人都可以是生产者，也可以是消费者

- **“不关心角色，只关注通道”**：CSP 的核心是**通道（Channel**本身，而非固定的“生产者”或“消费者”角色。
    
- **“动态角色分配”**：任何线程（或协程）都可以通过调用 `send()` 或 `receive()` 方法，**动态切换生产者或消费者身份**。
    
- **通道是唯一共享点**：所有通信必须通过通道完成，**没有直接的数据共享**。



是的，你提供的代码实现了一个简单的 **CSP（Communicating Sequential Processes）** 模型。通过使用 **Channel** 类，生产者和消费者线程可以通过通道进行通信，而无需直接共享数据。这种设计符合 CSP 的核心思想：**通过通信来共享内存，而不是通过共享内存来通信**。

---

### **代码分析**
#### **1. Channel 类的实现**
- **`queue_`**：用于存储数据的队列。
- **`mtx_`**：互斥锁，保护队列和条件变量的访问。
- **`cv_producer_`**：生产者条件变量，用于通知生产者是否可以发送数据。
- **`cv_consumer_`**：消费者条件变量，用于通知消费者是否可以接收数据。
- **`capacity_`**：通道的缓冲区大小（0 表示无缓冲通道）。
- **`closed_`**：标志位，表示通道是否已关闭。

#### **2. 关键方法**
- **`send(T value)`**：
  - 生产者调用此方法发送数据。
  - 如果通道已关闭，返回 `false`。
  - 如果通道无缓冲（`capacity_ == 0`），生产者需要等待消费者取走数据。
  - 如果通道有缓冲，生产者需要等待缓冲区未满。
  - 发送数据后，通知消费者。

- **`receive(T& value)`**：
  - 消费者调用此方法接收数据。
  - 如果通道已关闭且队列为空，返回 `false`。
  - 消费者需要等待队列非空。
  - 接收数据后，通知生产者。

- **`close()`**：
  - 关闭通道，唤醒所有等待的生产者和消费者。

#### **3. 示例使用**
- 生产者线程发送 5 个整数到通道。
- 消费者线程延迟 500 毫秒后开始接收数据。
- 通道关闭后，消费者线程退出。

---

### **CSP 的核心特性**
你的实现体现了 CSP 的以下核心特性：
1. **通信而非共享内存**：
   - 生产者和消费者通过 `Channel` 通信，而不是直接共享数据。
2. **同步通信**：
   - 无缓冲通道（`capacity_ == 0`）实现了同步通信，生产者必须等待消费者取走数据。
   - 有缓冲通道（`capacity_ > 0`）实现了异步通信，生产者可以在缓冲区未满时继续发送数据。
3. **线程安全**：
   - 使用互斥锁和条件变量确保线程安全。
4. **关闭机制**：
   - 通过 `close()` 方法优雅地关闭通道，避免死锁。

---

### **改进建议**
虽然你的实现已经很好地体现了 CSP 的思想，但仍有一些改进空间：
1. **无缓冲通道的实现**：
   - 当前代码中，无缓冲通道的逻辑可能不够清晰。可以明确区分无缓冲和有缓冲的行为：
     - 无缓冲通道：`send()` 必须等待 `receive()`，反之亦然。
     - 有缓冲通道：`send()` 可以在缓冲区未满时立即返回。
   - 修改 `send()` 中的条件：
     ```cpp
     cv_producer_.wait(lock, [this]() {
         return (capacity_ == 0 && queue_.empty()) || (capacity_ > 0 && queue_.size() < capacity_) || closed_;
     });
     ```

2. **异常处理**：
   - 在 `send()` 和 `receive()` 中添加异常处理，避免因通道关闭或其他问题导致程序崩溃。

3. **性能优化**：
   - 如果通道的容量较大，可以考虑使用环形缓冲区（`std::deque` 或自定义实现）代替 `std::queue`，以减少内存分配开销。

4. **更复杂的 CSP 模式**：
   - 支持多生产者和多消费者。
   - 实现超时机制（如 `try_send()` 和 `try_receive()`）。

---

### **示例改进代码**
以下是改进后的 `send()` 方法：
```cpp
bool send(T value) {
    std::unique_lock<std::mutex> lock(mtx_);
    cv_producer_.wait(lock, [this]() {
        // 无缓冲通道：等待队列为空
        // 有缓冲通道：等待队列未满
        return (capacity_ == 0 && queue_.empty()) || (capacity_ > 0 && queue_.size() < capacity_) || closed_;
    });

    if (closed_) {
        return false;
    }

    queue_.push(value);
    cv_consumer_.notify_one();
    return true;
}
```

---

### **总结**
你的代码已经实现了一个简单的 CSP 模型，符合通过通道通信的核心思想。通过进一步优化和完善，可以使其更健壮、更高效。CSP 模型在并发编程中非常有用，尤其是在需要解耦生产者和消费者的情况下。