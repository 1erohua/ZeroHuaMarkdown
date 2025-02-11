本节介绍C++线程管控，包括移交线程的归属权，线程并发数量控制以及获取线程id等基本操作。

# 使用容器存储线程
### 1. `emplace` 方式

`emplace` 是 C++11 引入的一种容器操作，用于在容器中直接构造元素，而不是先构造再拷贝或移动。它适用于 `std::vector`、`std::map`、`std::set` 等容器。

#### 1.1 `emplace_back` 与 `push_back` 的区别

- **`push_back`**: 接受一个已经构造好的对象，并将其拷贝或移动到容器中。
- **`emplace_back`**: 直接在容器中构造对象，避免了不必要的拷贝或移动操作。

#### 1.2 示例

```cpp
#include <vector>
#include <string>

int main() {
    std::vector<std::string> vec;

    // 使用 push_back
    vec.push_back(std::string("Hello"));

    // 使用 emplace_back
    vec.emplace_back("World");

    return 0;
}
```

- `push_back("Hello")` 会先构造一个 `std::string` 对象，再将其拷贝或移动到 `vec` 中。
- `emplace_back("World")` 则直接在 `vec` 中构造 `std::string` 对象，避免了额外的拷贝或移动。

### 2. 使用 `vector` 存储线程时 `push_back` 报错的原因

#### 2.1 `std::thread` 的特性

`std::thread` 是不可拷贝的，只能移动。这意味着你不能拷贝一个 `std::thread` 对象，但可以将其所有权转移到另一个 `std::thread` 对象。

#### 2.2 `push_back` 的问题

当你使用 `push_back` 向 `std::vector<std::thread>` 中添加线程时，`push_back` 会尝试拷贝或移动 `std::thread` 对象。由于 `std::thread` 不可拷贝，如果传递的是一个临时对象，编译器可能会尝试调用拷贝构造函数，从而导致编译错误。

#### 2.3 示例

```cpp
#include <vector>
#include <thread>

void task() {
    // 一些任务
}

int main() {
    std::vector<std::thread> threads;

    // 使用 push_back 会报错
    // threads.push_back(std::thread(task));  // 错误：std::thread 不可拷贝

    // 使用 emplace_back 正确
    threads.emplace_back(task);

    // 等待所有线程完成
    for (auto& t : threads) {
        t.join();
    }

    return 0;
}
```

- `push_back(std::thread(task))` 会尝试拷贝 `std::thread` 对象，导致编译错误。
- `emplace_back(task)` 直接在 `vector` 中构造 `std::thread` 对象，避免了拷贝操作。

### 3. 总结

- **`emplace_back`** 直接在容器中构造对象，避免了不必要的拷贝或移动操作，适用于不可拷贝的对象（如 `std::thread`）。
- **`push_back`** 需要对象已经构造好，并且要求对象可拷贝或移动，因此在存储 `std::thread` 时会导致编译错误。

因此，在使用 `std::vector` 存储 `std::thread` 时，应优先使用 `emplace_back` 来避免编译错误。

# 手写一份JoiningThread，在析构时自动join完成
```cpp
	// 个人感觉只要把析构函数的join改为detach，就能变为detaching_thread
class JoiningThread{
    std::thread _t;

private:
    // 默认构造函数仍然是默认
    JoiningThread() noexcept= default;

    // 模板构造，本质是使用thread自己的参数进行构造
    // 这里麻烦的是这一堆模板，但本质还是希望传给_t处理
    template<typename Callback, typename...Args>
    explicit JoiningThread(Callback&& func, Args&&... args):_t(std::forward<Callback>func, std::forward<Args>(args)...){}

    // 直接使用Thread对象进行构造————本质是调用的std::thread的移动构造函数
    // std::thread移动构造函数可以接受一个右值引用作为参数，并在内部实现了窃取资源的代码
    explicit JoiningThread(std::thread t)noexcept:_t(std::move(t)){};

    // 删除拷贝赋值和拷贝运算符
    JoiningThread(const JoiningThread& jt) = delete;
    JoiningThread& operator=(const JoiningThread& jt) = delete;

    // 实现移动构造与移动赋值
    // 同上，本质是调用thread的移动构造函数
    JoiningThread(JoiningThread&& jt)noexcept:_t(std::move(jt)){};

    JoiningThread& operator=(JoiningThread&& jt)noexcept{
        // 可汇合时，汇合完毕再转移
        if(joinable){
            join();
        }

        this->_t = std::move(jt._t);
        return *this;
    }

    // 一切的最终的目的————析构函数解决join问题
    ~JoiningThread(){
        if(joinable){
            join();
        }
    }

    // 剩下的就是包装thread了

    bool joinable()noexcept{
        return this->_t.joinable();
    }

    void join(){
        this->_t.join();
    }

    void detach(){
        _t.detach();
    }

    // swap，交换两个线程的句柄
    void swap(JoiningThread& jt){
        _t.swap(jt._t);
    }

    std::thread::id get_id()noexcept{
        return _t.get_id();
    }

    // 作为内部的thread返回
    std::thread& as_thread()noexcept{
        return _t;
    }

    const std::thread& as_thread() const noexcept{
        return _t;
    }
};
```


# emplace_back再学习
`std::emplace_back` 是 C++ 容器（如 `std::vector`、`std::deque` 等）的成员函数，用于直接在容器尾部构造对象，避免不必要的拷贝或移动操作。其实现和用途如下：

---

### **实现原理**
1. **可变参数模板（Variadic Templates）**  
   `emplace_back` 接受任意数量和类型的参数，通过模板参数推导匹配对象的构造函数。

2. **完美转发（Perfect Forwarding）**  
   使用 `std::forward` 将参数按原值类别（左值/右值）转发到对象的构造函数，确保参数传递的高效性。

3. **就地构造（In-Place Construction）**  
   在容器预分配的内存中直接调用对象的构造函数，无需创建临时对象。例如，`std::vector` 可能通过 `allocator_traits::construct` 或 `placement new` 在预留内存中构造对象。

---

### **用途**
1. **避免临时对象**  
   对于需要拷贝或移动的对象（如非平凡类型），`emplace_back` 直接通过参数构造元素，省去临时对象的创建和传递。

   ```cpp
   std::vector<std::string> vec;
   vec.emplace_back("Hello");  // 直接构造 std::string，无需创建临时对象
   ```

2. **支持多参数构造函数**  
   直接传递构造参数，无需手动创建对象或使用 `std::make_xxx`。

   ```cpp
   struct Foo { Foo(int a, int b) {} };
   std::vector<Foo> vec;
   vec.emplace_back(1, 2);  // 直接调用 Foo(int, int)
   ```

3. **隐式转换控制**  
   避免因隐式转换导致的意外行为，尤其当构造函数被 `explicit` 修饰时。

   ```cpp
   struct Bar { explicit Bar(int) {} };
   std::vector<Bar> vec;
   // vec.push_back(1);        // 错误：不能隐式转换 int 到 Bar
   vec.emplace_back(1);        // 正确：直接调用 Bar(int)
   ```

4. **性能优化**  
   对于构造代价高的对象（如大型数据结构或非移动类型），`emplace_back` 减少拷贝/移动开销。

---

### **与 `push_back` 的对比**
- **`push_back`**  
  需要先构造临时对象，再将其拷贝或移动到容器中：
  ```cpp
  vec.push_back(Foo(1, 2));  // 构造临时 Foo，再移动（或拷贝）到容器
  ```

- **`emplace_back`**  
  直接在容器内存中构造对象：
  ```cpp
  vec.emplace_back(1, 2);    // 无临时对象，直接在容器内构造 Foo(1,2)
  ```

---

### **注意事项**
- **参数顺序和类型**  
  需确保传递的参数与目标构造函数严格匹配，否则可能引发编译错误。

- **异常安全**  
  若构造过程中抛出异常，容器保持原有状态（强异常安全保证，前提是对象的构造函数不破坏不变式）。

- **返回值（C++17 起）**  
  C++17 后，`emplace_back` 返回新插入元素的引用；此前返回 `void`。

---

### **总结**
`std::emplace_back` 通过就地构造和完美转发优化容器元素添加过程，适用于需要高效构造对象或传递多参数的场景，是避免冗余拷贝/移动操作的推荐方式。