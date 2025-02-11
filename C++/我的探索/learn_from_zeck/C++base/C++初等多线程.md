# 初等多线程理解

**Happens-before** 是一个非常重要的概念. 如果操作 a “happens-before” 操作 b, 则操作 a 的结果对于操作 b 可见. happens-before 的关系可以建立在用一个线程的两个操作之间, 也可以建立在不同的线程的两个操作之间.

对于单线程，就是单纯的 **sequenced-before**，就是语句之间单纯的顺序

一般来说多线程都是并发执行的, 如果没有正确的同步操作, 就无法保证两个操作之间有 happens-before 的关系. 如果我们通过一些手段, 让不同线程的两个操作同步, 我们称这两个操作之间有 **synchronizes-with** 的关系
> 所谓synchronizes-with关系，就是两个不同线程会在某些操作上遵循特定的顺序。
> 我们都知道线程之间是异步操作，不同的线程有的执行快有的执行慢。使其中的某些步骤遵循必要的顺序，即同步，**即线程之间的同步操作**

如果线程 1 中的操作 a “synchronizes-with” 线程 2 中的操作 b, 则操作 a **“inter-thread happens-before”** 操作 b. 此外 synchronizes-with 还可以 “后接” 一个 sequenced-before 关系组合成 inter-thread happens-before 的关系:

- 如果操作 a “synchronizes-with” 操作 k, 且操作 k “sequenced-before” 操作 b, 则操作 a “inter-thread happens-before” 操作 b.
> 这段可能有点绕，耐心看，看得明白的

![[Pasted image 20250118164625.png]]

```cpp
void thread1() {  
// 一堆操作
a += 1 // (1)  
unlock(); // (2)  
}  
  
void thread2() {  
// 一堆操作
lock(); // (3)  
cout << a << endl; // (4)  
}
```
假设直到 `thread1` 执行到 (2) 之前, `thread2` 都会阻塞在 (3) 处的 `lock()` 中. 那么可以推导出:

- 根据语句顺序, 有 (1) 先于 (2)执行 ，且 (3) 先于 (4)执行;
- 因为  (3) 要等待（2）来解锁 ，且 (3) 先于 (4)执行, 所以 (2) “inter-thread happens-before (4);
- 因为 (1)先于(2)执行， 且 (2) “inter-thread happens-before” (4), 所以 (1) “inter-thread happens-before” (4); 所以 (1) “happens-before” (4).

因此 (4) 可以读到 (1) 对变量 `a` 的修改.

需要说明的是, happens-before 是 C++ 语义层面的概念, 它并不代表指令在 CPU 中实际的执行顺序. 为了优化性能, 编译器会在不破坏语义的前提下对指令重排. 例如

|                                      |                                                                             |
| ------------------------------------ | --------------------------------------------------------------------------- |
| 1  <br>2  <br>3  <br>4  <br>5  <br>6 | ```<br>extern int a, b;int add() {    a++;    b++;    return a + b;}<br>``` |

虽然有 `a++;` “happens-before” `b++;`, 但编译器实际生成的指令可能是先加载 `a`, `b` 两个变量到寄存器, 接着分别执行 “加一” 操作, 然后再执行 `a + b`, 最后才将自增的结果写入内存.

|                                                           |                                                                                                                                                                                                                                                                                                                  |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  <br>2  <br>3  <br>4  <br>5  <br>6  <br>7  <br>8  <br>9 | ```<br>add():    movl    a(%rip), %eax   # 将变量 a 加载到寄存器    movl    b(%rip), %ecx   # 将变量 b 加载到寄存器    addl    $1, %eax        # a 的值加一    leal    1(%rcx), %edx   # b 的值加一    movl    %eax, a(%rip)   # 将 a 加一的结果写入内存    addl    %edx, %eax      # a + b    movl    %edx, b(%rip)   # 将 b 加一的结果写入内存    ret<br>``` |

上面展示了 x86-64 下的一种可能的编译结果. 可以看到 C++ 的一条语句可能产生多条指令, 这些指令都是交错执行的. 其实编译器甚至还有可能先自增 `b` 再自增 `a`. 这样的重排并不会影响语义, 两个自增操作的结果仍然对 `return a + b;` 可见.

## 一个例子明白
```cpp
std::atomic<bool> x{false}, y{false};  
  
void thread1() {  
x.store(true, std::memory_order_relaxed); // (1)  
y.store(true, std::memory_order_relaxed); // (2)  
}

void thread2() {  
while (!y.load(std::memory_order_relaxed)); // (3)  
assert(x.load()); // (4)  
}
```
**原文内容**：thread1 对不同的变量执行 store 操作. 那么在某些线程看来, 有可能是 x 先变为 true, y 后变为 true; 另一些线程看来, 又有可能是 y 先变为 true, x 后变为 true.。
 
 (4) 处的断言就有可能失败. 因为 (2) 与 (3) 之间没有 synchronizes-with 的关系, 所以就不能保证 (1) “happens-before” (4). 因此 (4) 就有可能读到 false. 至于 relaxed 顺序模型能保证的修改顺序一致性的例子, 2.1 节中已经讨论过了, 这里就不多赘述了.


**我的疑问**：为什么1不是发生在4之前？在线程1中，x存完之后才能到y存，线程2中，while需要等待y存好才能读取x的值，这样为什么不是这样？


**答：**
你提到的逻辑是基于程序顺序（program order）的直觉，即在一个线程中，代码的执行顺序是按照代码的书写顺序进行的。然而，在多线程环境下，尤其是在使用 `memory_order_relaxed` 时，编译器和处理器可能会对指令进行重排序，这会导致不同线程看到的操作顺序不一致。

### 为什么 (1) 不一定是发生在 (4) 之前？

1. **`memory_order_relaxed` 的语义**：
   - `memory_order_relaxed` 只保证操作的原子性，不保证操作的顺序性。也就是说，它不会引入任何同步关系（synchronizes-with）或 happens-before 关系。
   - 在 `thread1` 中，(1) 和 (2) 是两个独立的 `store` 操作，使用 `memory_order_relaxed` 时，编译器和处理器可以自由地对这两个操作进行重排序。因此，其他线程可能会看到 (2) 先于 (1) 发生。

2. **线程间的可见性问题**：
   - 在 `thread2` 中，(3) 处的 `while` 循环会一直等待，直到 `y` 变为 `true`。然而，由于 `memory_order_relaxed` 不保证同步关系，`thread2` 可能会在 `y` 变为 `true` 之后立即读取 `x`，而此时 `x` 可能还没有被 `thread1` 设置为 `true`。
   - 这是因为 `memory_order_relaxed` 不保证 `thread1` 中的 (1) 和 (2) 操作的顺序对其他线程是可见的。`thread2` 可能会看到 `y` 变为 `true`，但 `x` 仍然是 `false`。

3. **没有 happens-before 关系**：
   - `happens-before` 关系是多线程程序中确保操作顺序的关键。在使用 `memory_order_relaxed` 时，(1) 和 (4) 之间没有建立任何 happens-before 关系。因此，(4) 处的 `assert` 可能会失败，因为 `x` 的值可能仍然是 `false`。

### 例子中的具体问题

- 在 `thread1` 中，(1) 和 (2) 是两个独立的 `store` 操作，使用 `memory_order_relaxed` 时，编译器和处理器可以自由地对这两个操作进行重排序。因此，其他线程可能会看到 (2) 先于 (1) 发生。
- 在 `thread2` 中，(3) 处的 `while` 循环会一直等待，直到 `y` 变为 `true`。然而，由于 `memory_order_relaxed` 不保证同步关系，`thread2` 可能会在 `y` 变为 `true` 之后立即读取 `x`，而此时 `x` 可能还没有被 `thread1` 设置为 `true`。

### 如何修复这个问题？

如果你希望确保 (1) 发生在 (4) 之前，可以使用更强的内存顺序，例如 `memory_order_release` 和 `memory_order_acquire`：

```cpp
void thread1() {
    x.store(true, std::memory_order_relaxed); // (1)
    y.store(true, std::memory_order_release); // (2)
}

void thread2() {
    while (!y.load(std::memory_order_acquire)); // (3)
    assert(x.load(std::memory_order_relaxed)); // (4)
}
```

在这个修改后的版本中：
- `y.store(true, std::memory_order_release)` 会确保在 (1) 和 (2) 之间建立一个 release 屏障，防止 (1) 被重排序到 (2) 之后。
- `y.load(std::memory_order_acquire)` 会确保在 (3) 和 (4) 之间建立一个 acquire 屏障，防止 (4) 被重排序到 (3) 之前。

这样，`thread2` 在看到 `y` 变为 `true` 之后，`x` 的值一定会是 `true`，从而避免了断言失败。

> 所谓内存顺序，其实就是编译器背锅。
> 编译器和处理器会重排序代码，换言之，你的代码运行顺序不是按照你写的那样来的，编译器和处理器会重拍。
> 
> 而指定内存顺序就是告诉编译器，别他妈乱动我的代码，至少这个要在那个之前运行，那边要在这边运行之后，诸如此类。


 [上述的参考文献，感谢](https://luyuhuang.tech/2022/06/25/cpp-memory-order.html#%E5%86%85%E5%AD%98%E9%A1%BA%E5%BA%8F)



#  std::atomic

好的！我将逐个详细解析 `std::atomic` 的常用方法，每个方法都会从 **功能**、**典型用法**、**底层原理** 和 **注意事项** 展开，并提供代码示例。以下是完整的解析：

---

## 1. **基础操作：`load()` 和 `store()`**

### **`T load(std::memory_order order = std::memory_order_seq_cst) const`**
- **功能**  
  原子地读取当前值，保证不会被其他线程的写入操作打断。
- **内存序**  
  默认严格顺序（`memory_order_seq_cst`），可通过参数调整。
- **示例**：
  ```cpp
  std::atomic<int> counter(0);
  int current = counter.load(); // 安全读取当前值
  ```

### **`void store(T desired, std::memory_order order = std::memory_order_seq_cst)`**
- **功能**  
  原子地写入新值，保证其他线程的读取操作会看到完整写入。
- **示例**：
  ```cpp
  std::atomic<bool> flag(false);
  flag.store(true); // 原子设置为 true
  ```

---

## 2. **交换操作：`exchange()`**

### **`T exchange(T desired, std::memory_order order = std::memory_order_seq_cst)`**
- **功能**  
  原子地将当前值替换为 `desired`，并返回旧值。
- **用途**  
  适用于需要“取出并替换”的场景，如简单锁实现。
- **示例**（自旋锁）：
  ```cpp
  std::atomic<bool> lock(false);

  void acquire_lock() {
      while (lock.exchange(true)) {} // 旧值为 true 表示锁已被占用，循环等待
  }

  void release_lock() {
      lock.store(false);
  }
  ```

---

## 3. **读-改-写操作：`fetch_add()` 和 `fetch_sub()`**

### **`T fetch_add(T arg, std::memory_order order = std::memory_order_seq_cst)`**
### **`T fetch_sub(T arg, std::memory_order order = std::memory_order_seq_cst)`**
- **功能**  
  原子地将当前值增加（或减少） `arg`，返回旧值。
- **底层实现**  
  等价于 `exchange(old_value + arg)`，但某些硬件支持更高效指令。
- **示例**（线程安全计数器）：
  ```cpp
  std::atomic<int> counter(0);

  void increment() {
      counter.fetch_add(1); // 原子递增
  }

  void decrement() {
      counter.fetch_sub(1); // 原子递减
  }
  ```

---

## 4. **位操作：`fetch_and()`, `fetch_or()`, `fetch_xor()`**

### **`T fetch_and(T arg, std::memory_order order = ...)`**
### **`T fetch_or(T arg, ...)`**
### **`T fetch_xor(T arg, ...)`**
- **功能**  
  原子地执行按位与、或、异或操作，返回旧值。
- **用途**  
  标志位管理、位掩码操作。
- **示例**（线程安全位标志）：
  ```cpp
  std::atomic<uint32_t> flags(0);

  void set_flag(uint32_t mask) {
      flags.fetch_or(mask); // 原子设置某一位
  }

  void clear_flag(uint32_t mask) {
      flags.fetch_and(~mask); // 原子清除某一位
  }
  ```

---

## 5. **比较并交换（CAS）：`compare_exchange_strong` 和 `compare_exchange_weak`**

### **`bool compare_exchange_strong(T& expected, T desired, ...)`**
- **核心原理**  
  检查原子变量当前值是否等于 `expected`：
  - 如果相等 → 替换为 `desired`，返回 `true`。
  - 如果不相等 → 将 `expected` 更新为实际值，返回 `false`。
- **典型模式**（循环重试）：
  ```cpp
  std::atomic<int> val(10);
  int expected = val.load();

  do {
      int desired = expected * 2;
  } while (!val.compare_exchange_strong(expected, desired));
  ```
- **注意事项**  
  必须基于最新的 `expected` 计算 `desired`，否则可能死循环。（确实会出现死循环）

### **`compare_exchange_weak` 的独特行为**
- **允许虚假失败**：即使 `expected` 等于实际值，也可能返回 `false`。
- **适用场景**  
  在循环中使用，通常比 `strong` 版本性能更高（尤其在 x86 平台）。
- **示例**：
  ```cpp
  do {
      // 每次循环必须重新计算 desired！
      desired = ...;
  } while (!val.compare_exchange_weak(expected, desired));
  ```

---

## 6. **指针特化操作：`fetch_add()` 和 `fetch_sub()`**

### **`仅适用于 `std::atomic<T*>`**
- **功能**  
  原子地调整指针的偏移量（单位为 `sizeof(T)`）。
- **示例**（线程安全的动态数组索引）：
  ```cpp
  struct Data { int x; };
  Data array[100];
  std::atomic<Data*> ptr(array);

  void append_data() {
      Data* old_ptr = ptr.load();
      Data* new_ptr = old_ptr + 1;
      while (!ptr.compare_exchange_weak(old_ptr, new_ptr)) {
          new_ptr = old_ptr + 1;
      }
      // 现在可以安全操作 old_ptr 指向的位置
  }
  ```

---

## 7. **内存序（Memory Order）参数详解**

每个原子操作可指定内存序，控制操作的可见性和顺序性：

| 内存序                    | 作用                                                                 |
|---------------------------|----------------------------------------------------------------------|
| `memory_order_relaxed`    | 仅保证原子性，无顺序约束（性能最高，适合计数器）                     |
| `memory_order_acquire`    | 当前操作的读取在后续操作前完成（用于“获取”锁）                       |
| `memory_order_release`    | 当前操作的写入在后续操作前完成（用于“释放”锁）                       |
| `memory_order_seq_cst`    | 严格全局顺序（默认选项，性能最低但最安全）                           |

- **示例**（使用宽松内存序的计数器）：
  ```cpp
  std::atomic<int> counter(0);
  counter.fetch_add(1, std::memory_order_relaxed); // 高性能但无顺序保证
  ```

---

## 8. **特殊类型：`std::atomic_flag`**

### **`bool test_and_set(std::memory_order order = ...)`**
### **`void clear(std::memory_order order = ...)`**
- **功能**  
  - `test_and_set()`：原子地将标志设为 `true`，返回旧值。
  - `clear()`：原子地将标志设为 `false`。
- **用途**  
  实现自旋锁（唯一保证无锁的原子类型）。
- **示例**：
  ```cpp
  std::atomic_flag lock = ATOMIC_FLAG_INIT;

  void critical_section() {
      while (lock.test_and_set()) {} // 获取锁
      // 临界区代码
      lock.clear(); // 释放锁
  }
  ```

---

## 关键问题解答

### **为什么 `compare_exchange` 需要循环？**
- 并发环境中，其他线程可能修改了原子变量。循环确保在失败时基于最新值重试，避免使用过期数据。

### **`compare_exchange_strong` 和 `weak` 如何选择？**
- 在循环中优先使用 `weak`（性能更高），单次尝试用 `strong`（避免虚假失败）。

### **内存序参数是否必须指定？**
- 默认使用 `memory_order_seq_cst`（安全但慢），高性能场景可调整（需深入理解内存模型）。

---

以上是 `std::atomic` 核心方法的详细解析。如果需要进一步探讨某个方法的底层硬件实现或复杂并发模式中的用法，请随时提出！

# std::mutex

### 2. `std::mutex`

`std::mutex` 是C++11引入的互斥量类，用于保护共享资源，防止多个线程同时访问。`std::mutex` 提供了基本的互斥锁功能，可以通过 `lock()` 和 `unlock()` 方法来手动加锁和解锁，也可以使用 `std::lock_guard` 或 `std::unique_lock` 来自动管理锁的生命周期。

#### 2.1 基本用法

```cpp
#include <mutex>
#include <iostream>
#include <thread>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();
        ++counter;
        mtx.unlock();
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Counter value: " << counter << std::endl;
    return 0;
}
```

在这个例子中，`mtx` 是一个 `std::mutex` 对象，`lock()` 和 `unlock()` 方法用于手动加锁和解锁。这样可以确保在 `++counter` 操作时，只有一个线程能够访问 `counter`。

#### 2.2 使用 `std::lock_guard`

为了避免手动调用 `lock()` 和 `unlock()`，可以使用 `std::lock_guard` 来自动管理锁的生命周期。

```cpp
#include <mutex>
#include <iostream>
#include <thread>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        ++counter;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Counter value: " << counter << std::endl;
    return 0;
}
```

`std::lock_guard` **在构造时自动加锁，在析构时自动解锁**，确保即使在发生异常的情况下，锁也能被正确释放。

#### 2.3 使用 `std::unique_lock`

`std::unique_lock` 比 `std::lock_guard` 更灵活，允许手动加锁和解锁，并且可以延迟加锁。

```cpp
#include <mutex>
#include <iostream>
#include <thread>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        ++counter;
        lock.unlock(); // 可以手动解锁
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);

    t1.join();
    t2.join();

    std::cout << "Counter value: " << counter << std::endl;
    return 0;
}
```

### 3. `std::atomic` 和 `std::mutex` 的区别

- **性能**：`std::atomic` 通常比 `std::mutex` 更高效，因为它直接使用硬件支持的原子操作，而不需要操作系统介入。
- **适用范围**：`std::atomic` 适用于简单的数据类型（如整数、指针等），而 `std::mutex` 适用于更复杂的场景，如保护复杂的数据结构或临界区。
- **灵活性**：`std::mutex` 提供了更多的灵活性，可以手动控制锁的粒度，而 `std::atomic` 的操作是原子的，无法进一步细分。

### 4. 总结

- 使用 `std::atomic` 时，适用于简单的数据类型和操作，性能较高。
- 使用 `std::mutex` 时，适用于复杂的临界区保护，灵活性更高。

在实际开发中，应根据具体需求选择合适的工具来确保线程安全。

是的，`std::unique_lock<std::mutex> lock(mtx)` 这行代码定义了一个 `std::unique_lock` 类型的对象 `lock`，并将 `mtx`（一个 `std::mutex` 对象）作为参数传递给它的构造函数。

### 详细解释

#### 1. `std::unique_lock` 是什么？
`std::unique_lock` 是C++标准库中的一个类模板，用于管理互斥锁（如 `std::mutex`）。它提供了比 `std::lock_guard` 更灵活的功能，例如：
- 手动加锁和解锁。
- 延迟加锁（即在构造时不立即加锁）。
- 支持条件变量（`std::condition_variable`）。

#### 2. 这行代码的作用
```cpp
std::unique_lock<std::mutex> lock(mtx);
```
- `std::unique_lock<std::mutex>`：定义了一个 `std::unique_lock` 对象，模板参数 `std::mutex` 表示它管理的锁类型是 `std::mutex`。
- `lock`：这是对象的名称，你可以根据需要命名。
- `mtx`：这是一个 `std::mutex` 对象，作为参数传递给 `std::unique_lock` 的构造函数。

#### 3. 构造函数的行为
当这行代码执行时，`std::unique_lock` 的构造函数会被调用，并完成以下操作：
- 自动对 `mtx` 加锁（除非显式指定延迟加锁）。
- 将 `mtx` 的所有权转移到 `lock` 对象中。

#### 4. 对象的生命周期
`lock` 是一个局部对象，它的生命周期受作用域控制：
- 当 `lock` 对象被创建时，`mtx` 会被加锁。
- 当 `lock` 对象离开作用域时（例如函数返回或代码块结束），它的析构函数会自动调用，释放 `mtx` 的锁。

#### 5. 示例代码
```cpp
#include <mutex>
#include <iostream>
#include <thread>

std::mutex mtx;

void critical_section() {
    std::unique_lock<std::mutex> lock(mtx); // 加锁
    std::cout << "Thread " << std::this_thread::get_id() << " is in the critical section.\n";
    // 这里可以安全地访问共享资源
} // lock 离开作用域，自动解锁

int main() {
    std::thread t1(critical_section);
    std::thread t2(critical_section);

    t1.join();
    t2.join();

    return 0;
}
```

#### 6. 与 `std::lock_guard` 的区别
- `std::lock_guard` 更简单，构造时加锁，析构时解锁，不能手动控制锁。
- `std::unique_lock` 更灵活，支持手动加锁和解锁，延迟加锁，以及条件变量。

#### 7. 延迟加锁的示例
```cpp
std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 构造时不加锁
lock.lock(); // 手动加锁
// 访问共享资源
lock.unlock(); // 手动解锁
```

### 总结
`std::unique_lock<std::mutex> lock(mtx)` 这行代码定义了一个 `std::unique_lock` 对象 `lock`，它在构造时自动对 `mtx` 加锁，并在析构时自动解锁。这种机制确保了在作用域内对共享资源的访问是线程安全的。

## 怎么延迟加锁
在C++中，`std::unique_lock` 提供了延迟加锁的功能，即可以在构造 `std::unique_lock` 对象时不立即加锁，而是在需要时手动加锁。这是通过向 `std::unique_lock` 的构造函数传递 `std::defer_lock` 标志来实现的。

### 延迟加锁的步骤

1. **构造 `std::unique_lock` 对象时传递 `std::defer_lock`**：
   这样会在构造时不加锁，而是将锁的管理权交给 `std::unique_lock` 对象。

2. **在需要时手动调用 `lock()` 方法**：
   当需要加锁时，可以显式调用 `lock()` 方法。

3. **在需要时手动调用 `unlock()` 方法**：
   如果希望在锁的作用域内提前释放锁，可以调用 `unlock()` 方法。

4. **析构时自动解锁**：
   如果锁仍然处于加锁状态，`std::unique_lock` 对象在析构时会自动解锁。

---

### 示例代码

以下是一个使用延迟加锁的完整示例：

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 定义一个互斥锁

void critical_section(int id) {
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 延迟加锁

    // 模拟一些不需要锁的操作
    std::cout << "Thread " << id << " is doing some non-critical work.\n";

    // 手动加锁
    std::cout << "Thread " << id << " is trying to lock the mutex.\n";
    lock.lock(); // 手动加锁
    std::cout << "Thread " << id << " has locked the mutex.\n";

    // 访问共享资源
    std::cout << "Thread " << id << " is in the critical section.\n";

    // 手动解锁（可选）
    lock.unlock();
    std::cout << "Thread " << id << " has unlocked the mutex.\n";

    // 锁会在 lock 对象析构时自动释放（如果仍然处于加锁状态）
}

int main() {
    std::thread t1(critical_section, 1);
    std::thread t2(critical_section, 2);

    t1.join();
    t2.join();

    return 0;
}
```

---

### 代码解析

1. **延迟加锁**：
   ```cpp
   std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
   ```
   - `std::defer_lock` 是一个标志，告诉 `std::unique_lock` 在构造时不加锁。
   - 此时 `mtx` 没有被加锁，`lock` 对象只是接管了 `mtx` 的管理权。

2. **手动加锁**：
   ```cpp
   lock.lock();
   ```
   - 在需要加锁时，显式调用 `lock()` 方法。
   - 此时 `mtx` 被加锁，其他线程无法获取锁。

3. **手动解锁**：
   ```cpp
   lock.unlock();
   ```
   - 如果希望在锁的作用域内提前释放锁，可以调用 `unlock()` 方法。
   - 解锁后，其他线程可以获取锁。

4. **自动解锁**：
   - 如果 `lock` 对象在析构时仍然处于加锁状态，它会自动调用 `unlock()` 释放锁。

---

### 延迟加锁的适用场景

1. **需要灵活控制锁的粒度**：
   - 如果某些操作不需要加锁，而只有部分操作需要加锁，可以使用延迟加锁来优化性能。

2. **需要手动控制锁的顺序**：
   - 在复杂的多线程场景中，可能需要手动控制多个锁的加锁顺序，以避免死锁。

3. **与条件变量一起使用**：
   - `std::unique_lock` 的延迟加锁功能常用于与 `std::condition_variable` 配合使用，以实现线程间的同步。

---

### 与 `std::lock_guard` 的对比

| 特性                | `std::lock_guard`                  | `std::unique_lock`                |
|---------------------|------------------------------------|------------------------------------|
| **加锁时机**         | 构造时立即加锁                     | 可以延迟加锁（通过 `std::defer_lock`） |
| **手动解锁**         | 不支持                             | 支持                               |
| **灵活性**           | 较低                               | 较高                               |
| **性能**             | 较高（无额外开销）                 | 较低（有额外状态管理开销）         |

---

### 总结

- 延迟加锁通过 `std::unique_lock` 和 `std::defer_lock` 实现。
- 它提供了更灵活的控制锁的方式，适用于需要手动控制加锁和解锁时机的场景。
- 如果不需要延迟加锁，可以使用 `std::lock_guard` 来简化代码。


## 锁与mutable
在C++中，`mutable` 关键字用于修饰类的成员变量，允许这些变量在 `const` 成员函数中被修改。锁（如 `std::mutex`）是否使用 `mutable` 取决于具体的使用场景和设计需求。

---

### 1. **什么时候使用 `mutable` 修饰锁？**

在多线程编程中，锁通常用于保护共享资源。如果一个类的成员函数是 `const` 的（即不会修改对象的状态），但仍然需要加锁来保证线程安全，那么锁就需要被声明为 `mutable`。

#### 示例：`mutable` 锁的使用

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class SharedData {
private:
    mutable std::mutex mtx; // 使用 mutable 修饰锁
    int value = 0;

public:
    void setValue(int val) {
        std::lock_guard<std::mutex> lock(mtx);
        value = val;
    }

    int getValue() const { // const 成员函数
        std::lock_guard<std::mutex> lock(mtx); // 需要加锁，但 mtx 是 mutable 的
        return value;
    }
};

int main() {
    SharedData data;
    data.setValue(42);

    std::thread t1([&data]() {
        std::cout << "Value: " << data.getValue() << std::endl;
    });

    t1.join();
    return 0;
}
```

#### 代码解析：
- `getValue()` 是一个 `const` 成员函数，它不会修改对象的状态。
- 但是，为了线程安全，`getValue()` 需要加锁来保护对 `value` 的访问。
- 由于 `mtx` 被声明为 `mutable`，因此可以在 `const` 成员函数中修改它（加锁和解锁）。

---

### 2. **什么时候不需要 `mutable` 修饰锁？**

如果锁不需要在 `const` 成员函数中使用，或者类的设计本身不涉及 `const` 成员函数，那么锁不需要被声明为 `mutable`。

#### 示例：不需要 `mutable` 的情况

```cpp
#include <iostream>
#include <mutex>
#include <thread>

class SharedData {
private:
    std::mutex mtx; // 不需要 mutable
    int value = 0;

public:
    void setValue(int val) {
        std::lock_guard<std::mutex> lock(mtx);
        value = val;
    }

    int getValue() { // 非 const 成员函数
        std::lock_guard<std::mutex> lock(mtx);
        return value;
    }
};

int main() {
    SharedData data;
    data.setValue(42);

    std::thread t1([&data]() {
        std::cout << "Value: " << data.getValue() << std::endl;
    });

    t1.join();
    return 0;
}
```

#### 代码解析：
- `getValue()` 不是 `const` 成员函数，因此可以直接使用普通的 `std::mutex`，而不需要 `mutable`。

---

### 3. **`mutable` 的使用场景总结**

| 场景                          | 是否需要 `mutable` |
|-------------------------------|--------------------|
| 锁在 `const` 成员函数中使用     | 需要               |
| 锁仅在非 `const` 成员函数中使用 | 不需要             |
| 锁用于保护逻辑上的常量状态      | 需要               |

---

### 4. **`mutable` 的注意事项**

1. **不要滥用 `mutable`**：
   - `mutable` 应该仅用于逻辑上不会改变对象状态的成员变量（如锁、缓存等）。
   - 滥用 `mutable` 可能导致代码难以理解和维护。

2. **锁的设计原则**：
   - 锁的作用是保护共享资源，而不是改变对象的状态。
   - 因此，即使锁在 `const` 成员函数中被修改，也不会违反 `const` 的语义。

3. **线程安全性**：
   - 即使锁是 `mutable` 的，也需要确保所有对共享资源的访问都是线程安全的。

---

### 5. **总结**

- 如果锁需要在 `const` 成员函数中使用，则应该将其声明为 `mutable`。
- 如果锁仅在非 `const` 成员函数中使用，则不需要 `mutable`。
- `mutable` 的使用应遵循最小化原则，避免滥用。

通过合理使用 `mutable`，可以在保证线程安全的同时，遵循 `const` 的正确语义。


# std::scoped_lock
`std::scoped_lock` 是 C++17 引入的一个 RAII（资源获取即初始化）风格的锁管理工具，用于管理多个互斥锁（`std::mutex`）的锁定和解锁操作。它的主要目的是简化多线程编程中的锁管理，避免死锁，并确保在作用域结束时自动释放锁。

### 1. 基本用法
`std::scoped_lock` 是一个模板类，可以接受一个或多个互斥锁类型作为模板参数。它的构造函数会锁定所有传入的互斥锁，并在析构时自动解锁这些锁。

#### 示例代码：
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mutex1;
std::mutex mutex2;

void thread_function() {
    std::scoped_lock lock(mutex1, mutex2); // 锁定 mutex1 和 mutex2
    // 临界区代码
    std::cout << "Thread ID: " << std::this_thread::get_id() << " is in the critical section.\n";
    // 锁在作用域结束时自动释放
}

int main() {
    std::thread t1(thread_function);
    std::thread t2(thread_function);

    t1.join();
    t2.join();

    return 0;
}
```

### 2. 避免死锁
`std::scoped_lock` 的一个重要特性是它能够避免死锁。当需要锁定多个互斥锁时，`std::scoped_lock` 会使用一种避免死锁的算法（通常是按照固定顺序锁定互斥锁），从而确保不会发生死锁。

#### 示例代码：
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mutex1;
std::mutex mutex2;

void thread_function1() {
    std::scoped_lock lock(mutex1, mutex2); // 锁定 mutex1 和 mutex2
    std::cout << "Thread 1 is in the critical section.\n";
}

void thread_function2() {
    std::scoped_lock lock(mutex2, mutex1); // 锁定 mutex2 和 mutex1
    std::cout << "Thread 2 is in the critical section.\n";
}

int main() {
    std::thread t1(thread_function1);
    std::thread t2(thread_function2);

    t1.join();
    t2.join();

    return 0;
}
```

在这个例子中，即使 `thread_function1` 和 `thread_function2` 以不同的顺序锁定互斥锁，`std::scoped_lock` 也会确保不会发生死锁。

### 3. 与 `std::lock_guard` 的区别
`std::scoped_lock` 是 `std::lock_guard` 的增强版。`std::lock_guard` 只能管理一个互斥锁，而 `std::scoped_lock` 可以管理多个互斥锁。此外，`std::scoped_lock` 还提供了避免死锁的机制。

#### 示例代码：
```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::mutex mutex1;
std::mutex mutex2;

void thread_function() {
    std::lock_guard<std::mutex> lock1(mutex1);
    std::lock_guard<std::mutex> lock2(mutex2);
    std::cout << "Thread ID: " << std::this_thread::get_id() << " is in the critical section.\n";
}

int main() {
    std::thread t1(thread_function);
    std::thread t2(thread_function);

    t1.join();
    t2.join();

    return 0;
}
```

在这个例子中，如果两个线程以不同的顺序锁定互斥锁，可能会导致死锁。而使用 `std::scoped_lock` 则可以避免这种情况。

### 4. 性能考虑
`std::scoped_lock` 的性能与 `std::lock_guard` 相当，但在管理多个互斥锁时，`std::scoped_lock` 会稍微增加一些开销，因为它需要使用避免死锁的算法。

### 5. 总结
- `std::scoped_lock` 是 C++17 引入的一个用于管理多个互斥锁的 RAII 工具。
- 它可以避免死锁，并确保在作用域结束时自动释放锁。
- 与 `std::lock_guard` 相比，`std::scoped_lock` 更强大，特别是在需要锁定多个互斥锁时。

使用 `std::scoped_lock` 可以简化多线程编程中的锁管理，减少出错的可能性，并提高代码的可读性和可维护性。

`std::scoped_lock` 是 C++17 引入的一个工具，用于同时锁定多个互斥量，并且能够避免死锁。它的核心机制是通过一种称为 **“死锁避免算法”**（Deadlock Avoidance Algorithm）的方式来确保所有线程以一致的顺序锁定互斥量，从而避免死锁。

---

### 为什么 `std::scoped_lock` 能够以固定顺序解决死锁问题？

#### 1. **内部使用 `std::lock`**
`std::scoped_lock` 的内部实现依赖于 `std::lock`，而 `std::lock` 使用了一种死锁避免算法来锁定多个互斥量。具体来说：
- `std::lock` 会以一种**不确定的顺序**尝试锁定所有传入的互斥量。
- 如果某个互斥量无法锁定（例如被其他线程持有），它会释放已经锁定的所有互斥量，然后重新尝试。
- 这个过程会一直重复，直到所有互斥量都被成功锁定。

通过这种方式，`std::lock` 能够避免死锁，因为它不会让线程一直持有部分锁并等待其他锁。

#### 2. **一致的锁定顺序**
虽然 `std::lock` 在内部尝试锁定互斥量的顺序是不确定的，但它会确保所有线程最终以**一致的顺序**锁定互斥量。这是因为：
- 所有线程都通过 `std::lock` 来锁定互斥量，而 `std::lock` 的实现保证了全局的一致性。
- 即使不同线程以不同的顺序调用 `std::scoped_lock`，`std::lock` 也会通过其算法调整锁的顺序，确保不会出现循环等待（即死锁的核心条件）。

---

### 示例分析

假设有两个线程，分别执行以下代码：

```cpp
// 线程1
std::scoped_lock lock(mtx, other.mtx);

// 线程2
std::scoped_lock lock(other.mtx, mtx);
```

#### 没有 `std::scoped_lock` 的情况
如果没有 `std::scoped_lock`，线程1和线程2可能会以不同的顺序锁定互斥量：
- 线程1先锁定 `mtx`，然后尝试锁定 `other.mtx`。
- 线程2先锁定 `other.mtx`，然后尝试锁定 `mtx`。
这会导致死锁，因为两个线程互相等待对方释放锁。

#### 使用 `std::scoped_lock` 的情况
使用 `std::scoped_lock` 时：
1. 线程1和线程2都会调用 `std::scoped_lock`，并传入需要锁定的互斥量。
2. `std::scoped_lock` 内部会调用 `std::lock`，`std::lock` 会以一种避免死锁的方式锁定所有互斥量。
3. 无论线程1和线程2传入互斥量的顺序如何，`std::lock` 都会确保所有线程以一致的顺序锁定互斥量。

例如：
- `std::lock` 可能会决定先锁定 `mtx`，再锁定 `other.mtx`。
- 即使线程2传入的顺序是 `other.mtx, mtx`，`std::lock` 也会调整顺序，确保先锁定 `mtx`，再锁定 `other.mtx`。

---

### `std::scoped_lock` 的工作原理

以下是 `std::scoped_lock` 的简化工作流程：
1. 接收需要锁定的互斥量列表。
2. 调用 `std::lock` 来锁定所有互斥量，确保不会发生死锁。
3. 在 `std::scoped_lock` 对象析构时，自动释放所有互斥量（释放顺序与锁定顺序相反）。

---

### 总结

`std::scoped_lock` 能够以固定顺序解决死锁问题，是因为：
1. 它内部使用 `std::lock`，而 `std::lock` 实现了死锁避免算法。
2. `std::lock` 会确保所有线程以一致的顺序锁定互斥量，即使不同线程传入的顺序不同。
3. 通过这种方式，`std::scoped_lock` 避免了循环等待，从而解决了死锁问题。

因此，`std::scoped_lock` 是处理多个互斥量时的推荐工具，既简单又安全。