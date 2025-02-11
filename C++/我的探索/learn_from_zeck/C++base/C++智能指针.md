### C++ 中的智能指针

智能指针是 C++ 中用于管理动态内存的工具，能够自动释放内存，避免内存泄漏。C++ 标准库提供了几种智能指针：`std::unique_ptr`、`std::shared_ptr` 和 `std::weak_ptr`。

### 1. 智能指针的原理机制

智能指针的核心是通过 RAII（Resource Acquisition Is Initialization）机制，将资源的生命周期与对象的生命周期绑定。当智能指针对象销毁时，会自动释放其管理的资源。

#### 1.1 `std::unique_ptr`

`std::unique_ptr` 独占资源的所有权，不允许复制，但可以通过 `std::move` 转移所有权。

- **实现原理**：`std::unique_ptr` 内部通常包含一个原始指针，并在析构时调用 `delete` 或 `delete[]` 释放资源。
- **汇编本质**：在汇编层面，`std::unique_ptr` 的析构函数会在对象销毁时插入 `delete` 操作。

#### 1.2 `std::shared_ptr`

`std::shared_ptr` 通过引用计数管理资源，多个 `std::shared_ptr` 可以共享同一资源，引用计数为 0 时释放资源。

- **实现原理**：`std::shared_ptr` 内部包含一个指向控制块的指针，控制块存储引用计数和资源指针。引用计数通过原子操作保证线程安全。
- **汇编本质**：在汇编层面，`std::shared_ptr` 的构造和析构函数会更新引用计数，并在计数为 0 时调用 `delete`。

#### 1.3 `std::weak_ptr`

`std::weak_ptr` 是 `std::shared_ptr` 的弱引用，不增加引用计数，用于解决循环引用问题。

- **实现原理**：`std::weak_ptr` 内部也指向控制块，但不影响引用计数。通过 `lock()` 方法可以获取一个 `std::shared_ptr`。
- **汇编本质**：在汇编层面，`std::weak_ptr` 的操作主要涉及控制块的访问和 `lock()` 方法的调用。

### 2. 智能指针的用法

#### 2.1 `std::unique_ptr`

```cpp
#include <memory>

void example_unique_ptr() {
    std::unique_ptr<int> ptr(new int(10));
    // ptr 独占资源，不能复制
    std::unique_ptr<int> ptr2 = std::move(ptr); // 转移所有权
}
```
独占指针只能转移所有权，不能复制。
#### 2.2 `std::shared_ptr`

```cpp
#include <memory>

void example_shared_ptr() {
    std::shared_ptr<int> ptr1(new int(10));
    std::shared_ptr<int> ptr2 = ptr1; // 共享资源，引用计数增加
}
```
应该是重写了赋值函数，使得计数增加了
#### 2.3 `std::weak_ptr`

```cpp
#include <memory>

void example_weak_ptr() {
    std::shared_ptr<int> sharedPtr(new int(10));
    std::weak_ptr<int> weakPtr = sharedPtr; // 弱引用，不增加引用计数

    if (auto spt = weakPtr.lock()) { // 尝试获取 shared_ptr
        // 使用 spt
    }
}
```

### 3. 智能指针在汇编中的本质

智能指针在汇编层面的实现主要依赖于编译器生成的代码，包括构造函数、析构函数和赋值操作符。

#### 3.1 `std::unique_ptr`

```assembly
; 伪汇编代码
unique_ptr_constructor:
    mov eax, [new_ptr] ; 将新指针存入 eax
    mov [this], eax    ; 将 eax 存入 this 指针

unique_ptr_destructor:
    mov eax, [this]    ; 将 this 指针存入 eax
    delete eax         ; 调用 delete 释放内存
```

#### 3.2 `std::shared_ptr`

```assembly
; 伪汇编代码
shared_ptr_constructor:
    mov eax, [new_ptr] ; 将新指针存入 eax
    mov [this], eax    ; 将 eax 存入 this 指针
    atomic_inc [ref_count] ; 原子操作增加引用计数

shared_ptr_destructor:
    atomic_dec [ref_count] ; 原子操作减少引用计数
    cmp [ref_count], 0
    jne not_zero
    delete eax         ; 引用计数为 0，调用 delete 释放内存
not_zero:
    ; 其他操作
```

### 总结

智能指针通过 RAII 机制自动管理资源，避免内存泄漏。`std::unique_ptr` 独占资源，`std::shared_ptr` 通过引用计数共享资源，`std::weak_ptr` 提供弱引用。在汇编层面，智能指针的实现依赖于编译器生成的构造函数、析构函数和赋值操作符，确保资源的正确管理。



# std::shared_ptr与std::unique_ptr简单实现
```C++
#include <iostream>
#include <string>

template <typename T>
class SharePointer {
private:
    int* counter;
    T* pointer;

    void release() {
        if (counter) {
            (*counter)--;
            std::cout << "解除一个使用者" << std::endl;
            if (*counter == 0) {
                delete pointer;
                delete counter;
                std::cout << "该资源被彻底销毁" << std::endl;
            }
        }
        counter = nullptr;
        pointer = nullptr;
    }

public:
    SharePointer() : counter(nullptr), pointer(nullptr) {}

    explicit SharePointer(T* ptr) : pointer(ptr) {
        if (pointer) {
            counter = new int(1);
        } else {
            counter = nullptr;
        }
    }

    ~SharePointer() {
        release();
    }

    // 移动构造函数
    SharePointer(SharePointer&& sp) noexcept : counter(sp.counter), pointer(sp.pointer) {
        sp.counter = nullptr;
        sp.pointer = nullptr;
    }

    // 移动赋值操作符
    SharePointer& operator=(SharePointer&& sp) noexcept {
        if (this != &sp) {
            release();
            counter = sp.counter;
            pointer = sp.pointer;
            sp.counter = nullptr;
            sp.pointer = nullptr;
        }
        return *this;
    }

    // 拷贝构造函数
    SharePointer(const SharePointer& sp) : counter(sp.counter), pointer(sp.pointer) {
        if (counter) {
            (*counter)++;
        }
    }

    // 拷贝赋值操作符
    SharePointer& operator=(const SharePointer& sp) {
        if (this != &sp) {
            release();
            counter = sp.counter;
            pointer = sp.pointer;
            if (counter) {
                (*counter)++;
            }
        }
        return *this;
    }

    // 重载 * 运算符
    T& operator*() const {
        return *pointer;
    }

    // 重载 -> 运算符
    T* operator->() const {
        return pointer;
    }

    // 获取引用计数
    int use_count() const {
        return counter ? *counter : 0;
    }
};


template <typename T>
class UniquePtr {
private:
    T* ptr;

public:
    UniquePtr() : ptr(nullptr) {}

    explicit UniquePtr(T* p) : ptr(p) {}

    ~UniquePtr() {
        if (ptr) {
            delete ptr;
            std::cout << "资源已释放" << std::endl;
        }
    }

    // 删除拷贝构造函数和拷贝赋值操作符
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr& operator=(const UniquePtr&) = delete;

    // 移动构造函数
    UniquePtr(UniquePtr&& up) noexcept : ptr(up.ptr) {
        up.ptr = nullptr;
        std::cout << "移动构造完成" << std::endl;
    }

    // 移动赋值操作符
    UniquePtr& operator=(UniquePtr&& up) noexcept {
        if (this != &up) {
            if (ptr) {
                delete ptr;
            }
            ptr = up.ptr;
            up.ptr = nullptr;
            std::cout << "移动赋值完成" << std::endl;
        }
        return *this;
    }

    // 重载 * 运算符
    T& operator*() const {
        return *ptr;
    }

    // 重载 -> 运算符
    T* operator->() const {
        return ptr;
    }

    // 获取原始指针
    T* get() const {
        return ptr;
    }

    // 释放资源并返回原始指针
    T* release() {
        T* temp = ptr;
        ptr = nullptr;
        return temp;
    }

    // 重置指针
    void reset(T* p = nullptr) {
        if (ptr) {
            delete ptr;
        }
        ptr = p;
    }
};


int main() {
    // 共享指针测试
    int* p = new int(11);
    SharePointer<int> sp1(p);
    SharePointer<int> sp2 = sp1;
    SharePointer<int> sp3 = std::move(sp2);

    std::cout << "sp1 use_count: " << sp1.use_count() << std::endl;
    std::cout << "sp3 use_count: " << sp3.use_count() << std::endl;
    std::cout << "sp3 value: " << *sp3 << std::endl;

    // 独占指针测试
    double* pb = new double(11.21311);
    UniquePtr<double> up1(pb);
    UniquePtr<double> up2 = std::move(up1);

    std::cout << "up2 value: " << *up2 << std::endl;

    return 0;
}

```
# 上述实现代码的末端


# std::weak_ptr再详细解释
`std::weak_ptr` 是 C++ 标准库中的一种智能指针，用于解决 `std::shared_ptr` 的循环引用问题。它本身不拥有对象的所有权，而是观察一个由 `std::shared_ptr` 管理的对象。下面详细讲解 `std::weak_ptr` 的实现原理、存在的意义、解决的问题以及与其他指针的对比。

---

### 1. **`std::weak_ptr` 的实现原理**
`std::weak_ptr` 的核心原理是**弱引用**。它不增加所管理对象的引用计数，因此不会影响对象的生命周期。它的实现依赖于 `std::shared_ptr` 的控制块（control block）。

- **控制块**：`std::shared_ptr` 在管理对象时，会创建一个控制块，其中包含两个引用计数：
  - **强引用计数**：记录有多少个 `std::shared_ptr` 指向对象。
  - **弱引用计数**：记录有多少个 `std::weak_ptr` 指向对象。

- **弱引用计数的作用**：
  - 当强引用计数为 0 时，对象会被销毁，但控制块不会被释放，直到弱引用计数也为 0。
  - `std::weak_ptr` 可以通过 `lock()` 方法尝试提升为 `std::shared_ptr`，如果对象仍然存在，则增加强引用计数；否则返回空的 `std::shared_ptr`。

---

### 2. **为什么需要 `std::weak_ptr`**
`std::weak_ptr` 的主要作用是解决 `std::shared_ptr` 的循环引用问题。

#### 循环引用问题
循环引用是指两个或多个对象通过 `std::shared_ptr` 相互引用，导致它们的引用计数永远不会降为 0，从而无法释放内存。例如：

```cpp
class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::shared_ptr<A> a_ptr;
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>(); 
    // a是指向A类的某个对象的share指针，初始化后，指向A对象的指针计数+1, A对象存放着一个智能指针
    
    auto b = std::make_shared<B>(); 
    // b是指向B类的某个对象的share指针，初始化后，指向B对象的指针计数+1，B对象存放着另一个智能指针
    
    a->b_ptr = b; 
	// 指向B对象的指针数+1（赋值运算符） 
    
    
    b->a_ptr = a;
    // 指向A对象的指针数+1
    // 循环引用导致 A 和 B 无法被销毁

	// 是这样的
	// a被销毁时，a指向的A类对象没有被销毁，因而a->b_ptr这个东西仍然存在(b_ptr不是a的成员！！！)。b也同理

	// 这个复杂的关系是：
	// a指向A对象，A对象有个成员指向B对象
	// b指向B对象，B对象有个成员指向A对象
	// 其中分别有四个智能指针，a、b本身就是智能指针；A、B类对象各自管理着一个智能指针
	// 这里的问题就在于，a、b是分配在栈里面的，进程结束自动销毁；A、B分配在堆中，进程结束后不会销毁
	// 因而a、b销毁时，A、B的引用计数都为1,因为A、B自身的成员智能指针还指着对方，导致这两个智能指针都无法释放双方对象
	// 回看自己实现的SharePtr，智能指针引用计数为0的时候，才会释放指向的内存，即销毁A、B类对象
}
```

太变态了
在上面的代码中，`A` 和 `B` 相互持有对方的 `std::shared_ptr`，导致它们的引用计数始终为 1，无法被销毁。

#### 使用 `std::weak_ptr` 解决循环引用
将其中一个 `std::shared_ptr` 替换为 `std::weak_ptr`，可以打破循环引用：

```cpp
class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::weak_ptr<A> a_ptr; // 使用 weak_ptr 代替 shared_ptr
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>(); // A的引用计数+1
    auto b = std::make_shared<B>(); // B的引用计数+1
    a->b_ptr = b; // A 持有 B 的 shared_ptr // B的引用计数+1
    b->a_ptr = a; // B 持有 A 的 weak_ptr  // A的引用计数不变
    // 循环引用被打破，A 和 B 可以被正确销毁
    
    // 销毁时，先销毁a，a被销毁，则A的引用计数归0
    // A的引用计数归零，原本指向A的智能指针会在归零时释放A的内存，从而使得A被销毁
    // A被销毁了，原来A内的所有成员都会调用析构函数与A自身的析构函数，因而b_ptr被销毁，调用自己的析构函数，从而B的引用计数-1
    // b被销毁，B的引用计数再-1

}
```

此时，`B` 中的 `a_ptr` 是 `std::weak_ptr`，不会增加 `A` 的引用计数。当 `a` 超出作用域时，`A` 的引用计数降为 0，`A` 被销毁，随后 `B` 的引用计数也降为 0，`B` 被销毁。

---

### 3. **`std::weak_ptr` 解决了什么问题**
- **循环引用问题**：如上所述，`std::weak_ptr` 可以打破 `std::shared_ptr` 的循环引用，避免内存泄漏。
- **临时访问共享资源**：`std::weak_ptr` 允许临时访问由 `std::shared_ptr` 管理的资源，而不影响资源的生命周期。

---

### 4. **`std::weak_ptr` 与其他指针的对比**

| 特性                | `std::weak_ptr`                          | `std::shared_ptr`                     | `std::unique_ptr`                     | 原始指针 (`T*`)                       |
|---------------------|------------------------------------------|---------------------------------------|---------------------------------------|---------------------------------------|
| **所有权**          | 无所有权（弱引用）                       | 共享所有权                            | 独占所有权                            | 无所有权                              |
| **是否影响生命周期**| 不影响                                  | 影响                                  | 影响                                  | 不影响                                |
| **是否可复制**      | 是                                      | 是                                    | 否                                    | 是                                    |
| **是否可移动**      | 是                                      | 是                                    | 是                                    | 是                                    |
| **是否可空**        | 是                                      | 是                                    | 是                                    | 是                                    |
| **访问对象**        | 通过 `lock()` 提升为 `std::shared_ptr`   | 直接访问                              | 直接访问                              | 直接访问                              |
| **典型用途**        | 打破循环引用、临时访问共享资源           | 共享资源管理                          | 独占资源管理                          | 非拥有关系的对象访问                  |



---

### 5. **`std::weak_ptr` 的使用示例**

#### 示例 1：打破循环引用
```cpp
class Node {
public:
    std::weak_ptr<Node> next; // 使用 weak_ptr 避免循环引用
    ~Node() { std::cout << "Node destroyed\n"; }
};

int main() {
    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    node1->next = node2;
    node2->next = node1;
    // node1 和 node2 可以被正确销毁
}
```

#### 示例 2：临时访问共享资源
```cpp
void accessResource(std::weak_ptr<int> weak_ptr) {
    if (auto shared_ptr = weak_ptr.lock()) {
        std::cout << "Resource value: " << *shared_ptr << "\n";
    } else {
        std::cout << "Resource is no longer available\n";
    }
}

int main() {
    auto shared_ptr = std::make_shared<int>(42);
    std::weak_ptr<int> weak_ptr = shared_ptr;
    accessResource(weak_ptr); // 输出: Resource value: 42
    shared_ptr.reset(); // 释放资源
    accessResource(weak_ptr); // 输出: Resource is no longer available
}
```

---

### 6. **总结**
- `std::weak_ptr` 是一种弱引用指针，不拥有对象的所有权，主要用于解决 `std::shared_ptr` 的循环引用问题。
- 它通过 `lock()` 方法可以临时提升为 `std::shared_ptr`，以安全地访问资源。
- 与 `std::shared_ptr` 和 `std::unique_ptr` 相比，`std::weak_ptr` 更适用于需要观察资源但不影响其生命周期的场景。

通过合理使用 `std::weak_ptr`，可以避免内存泄漏，并编写出更健壮的 C++ 代码。




#  什么叫weak_ptr不能直接访问对象？
`std::weak_ptr` 是一种弱引用指针，它本身**不拥有对象的所有权**，因此不能直接访问所指向的对象。要访问 `std::weak_ptr` 所观察的对象，必须先将它转换为 `std::shared_ptr`，然后通过 `std::shared_ptr` 访问对象。这种设计是为了确保在访问对象时，对象仍然存在（即未被销毁）。

下面通过一个例子详细说明 `std::weak_ptr` 不能直接访问对象的原因以及如何正确使用它。

---

### 1. **`std::weak_ptr` 不能直接访问对象的原因**
- `std::weak_ptr` 是一种弱引用，它不会增加对象的引用计数。
- 由于它不拥有对象的所有权，无法保证所指向的对象仍然存在（可能已经被销毁）。
- 如果允许直接访问对象，可能会导致访问已释放的内存，引发未定义行为（如程序崩溃）。

---

### 2. **如何通过 `std::weak_ptr` 访问对象**
`std::weak_ptr` 提供了一个 `lock()` 方法，用于尝试将 `std::weak_ptr` 提升为 `std::shared_ptr`。如果对象仍然存在，`lock()` 会返回一个有效的 `std::shared_ptr`，否则返回一个空的 `std::shared_ptr`。

---

### 3. **示例代码**

#### 示例 1：直接访问 `std::weak_ptr` 会导致编译错误
```cpp
#include <iostream>
#include <memory>

int main() {
    auto shared_ptr = std::make_shared<int>(42);
    std::weak_ptr<int> weak_ptr = shared_ptr;

    // 错误：std::weak_ptr 不能直接访问对象
    // std::cout << *weak_ptr << "\n"; // 编译错误

    // 正确方式：使用 lock() 提升为 std::shared_ptr
    if (auto shared_ptr2 = weak_ptr.lock()) {
        std::cout << "Value: " << *shared_ptr2 << "\n"; // 输出: Value: 42
    } else {
        std::cout << "Object is no longer available\n";
    }

    // 释放 shared_ptr
    shared_ptr.reset();

    // 再次尝试访问
    if (auto shared_ptr2 = weak_ptr.lock()) {
        std::cout << "Value: " << *shared_ptr2 << "\n";
    } else {
        std::cout << "Object is no longer available\n"; // 输出: Object is no longer available
    }
}
```

#### 示例 2：`std::weak_ptr` 的典型使用场景
```cpp
#include <iostream>
#include <memory>

class Resource {
public:
    Resource() { std::cout << "Resource created\n"; }
    ~Resource() { std::cout << "Resource destroyed\n"; }
    void use() { std::cout << "Resource is being used\n"; }
};

int main() {
    std::weak_ptr<Resource> weak_ptr;

    {
        // 创建一个 Resource 对象，并由 shared_ptr 管理
        auto shared_ptr = std::make_shared<Resource>();
        weak_ptr = shared_ptr; // weak_ptr 观察 shared_ptr

        // 通过 weak_ptr 访问对象
        if (auto shared_ptr2 = weak_ptr.lock()) {
            shared_ptr2->use(); // 输出: Resource is being used
        } else {
            std::cout << "Resource is no longer available\n";
        }
    } // shared_ptr 超出作用域，Resource 对象被销毁

    // 再次尝试访问
    if (auto shared_ptr2 = weak_ptr.lock()) {
        shared_ptr2->use();
    } else {
        std::cout << "Resource is no longer available\n"; // 输出: Resource is no longer available
    }
}
```

---

### 4. **代码解析**

#### （1）`std::weak_ptr` 不能直接访问对象
在示例 1 中，尝试直接解引用 `std::weak_ptr` 会导致编译错误：
```cpp
std::cout << *weak_ptr << "\n"; // 编译错误
```
这是因为 `std::weak_ptr` 没有重载 `operator*` 或 `operator->`，无法直接访问对象。

#### （2）使用 `lock()` 方法安全访问对象
`lock()` 方法会尝试将 `std::weak_ptr` 提升为 `std::shared_ptr`：
- 如果对象仍然存在（即强引用计数 > 0），`lock()` 返回一个有效的 `std::shared_ptr`，可以安全地访问对象。
- 如果对象已被销毁（即强引用计数 = 0），`lock()` 返回一个空的 `std::shared_ptr`。

在示例 2 中：
```cpp
if (auto shared_ptr2 = weak_ptr.lock()) {
    shared_ptr2->use(); // 安全访问对象
} else {
    std::cout << "Resource is no longer available\n";
}
```

#### （3）`std::weak_ptr` 的典型使用场景
- **打破循环引用**：在存在循环引用的情况下，使用 `std::weak_ptr` 可以避免内存泄漏。
- **临时访问共享资源**：当需要临时访问由 `std::shared_ptr` 管理的资源时，可以使用 `std::weak_ptr` 观察资源，而不影响资源的生命周期。

---

### 5. **总结**
- `std::weak_ptr` 不能直接访问对象，因为它不拥有对象的所有权，无法保证对象仍然存在。
- 要通过 `std::weak_ptr` 访问对象，必须使用 `lock()` 方法将其提升为 `std::shared_ptr`。
- `lock()` 方法提供了一种安全的方式来访问对象，避免了访问已释放内存的风险。
- `std::weak_ptr` 的主要用途是打破循环引用和临时观察共享资源。

通过合理使用 `std::weak_ptr`，可以编写出更安全、更健壮的 C++ 代码。



# 智能指针的构造参数
在C++中，智能指针（如 `std::unique_ptr` 和 `std::shared_ptr`）可以接受多种参数来初始化。以下是常见的参数类型：

### 1. **裸指针**
   智能指针可以接受一个裸指针作为构造参数，表示它将管理该指针指向的对象。

   ```cpp
   std::unique_ptr<int> ptr(new int(42));
   ```

### 2. **自定义删除器**
   智能指针可以接受一个函数对象（如 lambda、函数指针或 `std::function`）作为删除器，用于自定义对象的销毁方式。

   ```cpp
   auto deleter = [](int* p) { delete p; };
   std::unique_ptr<int, decltype(deleter)> ptr(new int(42), deleter);
   ```

   对于 `std::shared_ptr`，删除器可以通过构造函数传递：

   ```cpp
   std::shared_ptr<int> ptr(new int(42), [](int* p) { delete p; });
   ```

### 3. **空指针**
   智能指针可以接受 `nullptr`，表示初始化为空指针。

   ```cpp
   std::unique_ptr<int> ptr(nullptr);
   ```

### 4. **移动语义**
   智能指针可以通过移动语义从另一个智能指针接管资源。

   ```cpp
   std::unique_ptr<int> ptr1(new int(42));
   std::unique_ptr<int> ptr2 = std::move(ptr1); // ptr1 现在为空
   ```

### 5. **`std::make_unique` 和 `std::make_shared`**
   这些函数可以接受构造对象所需的参数，并返回一个智能指针。

   ```cpp
   auto ptr = std::make_unique<int>(42);
   auto ptr2 = std::make_shared<std::string>("Hello");
   ```

### 6. **`std::shared_ptr` 的别名构造函数**
   `std::shared_ptr` 可以接受另一个 `std::shared_ptr` 和一个指针，用于创建别名指针。

   ```cpp
   std::shared_ptr<int> ptr1 = std::make_shared<int>(42);
   std::shared_ptr<int> ptr2(ptr1, ptr1.get()); // ptr2 是 ptr1 的别名
   ```

### 7. **`std::weak_ptr`**
   `std::shared_ptr` 可以接受 `std::weak_ptr` 来初始化，前提是 `weak_ptr` 尚未过期。

   ```cpp
   std::shared_ptr<int> ptr1 = std::make_shared<int>(42);
   std::weak_ptr<int> weakPtr = ptr1;
   std::shared_ptr<int> ptr2 = weakPtr.lock(); // 如果 weakPtr 未过期，ptr2 将管理资源
   ```

### 总结
C++ 智能指针的构造函数非常灵活，可以接受裸指针、删除器、空指针、移动语义、`std::make_unique`/`std::make_shared` 的参数、别名构造函数以及 `std::weak_ptr` 等。


# 智能指针的最佳实践与常见陷阱

### 7.1 选择合适的智能指针

- **`std::unique_ptr`**：
    - 用于明确的独占所有权场景。
    - 适用于资源的单一管理者或需要所有权转移的情况。
    - 更轻量，性能更优。
- **`std::shared_ptr`**：
    - 用于共享所有权的场景。
    - 需要多个指针共同管理同一资源时使用。
    - 引用计数带来一定的性能开销。
- **`std::weak_ptr`**：
    - 用于观察不拥有资源的场景。
    - 适用于需要避免循环引用或只需临时访问资源的情况。

### 7.2 避免循环引用

在使用 `shared_ptr` 时，特别是在对象间存在双向引用时，容易导致循环引用，内存泄漏。使用 `weak_ptr` 打破循环引用。

### 7.3 使用 `make_shared` 与 `make_unique`

优先使用 `make_shared` 和 `make_unique` 来创建智能指针，避免直接使用 `new`，提高效率和异常安全性。

```cpp
auto sp = std::make_shared<Test>(100);
auto up = std::make_unique<Test>(200);
```

### 7.4 不要混用原生指针与智能指针

避免在智能指针管理的对象上同时使用原生指针进行管理，防止重复释放或不安全访问。

### 7.5 理解智能指针的所有权语义

深入理解不同智能指针的所有权规则，避免误用导致资源管理错误。

## 8. 总结

智能指针是 C++ 中强大的资源管理工具，通过封装原生指针，提供自动化的内存管理，极大地减少了内存泄漏和资源管理错误。`std::unique_ptr`、`std::shared_ptr` 和 `std::weak_ptr` 各有其应用场景，理解它们的差异和使用方法对于编写安全、高效的 C++ 代码至关重要。此外，通过实现自己的智能指针（如 `SimpleSharedPtr`），可以更深入地理解智能指针的工作原理，为高级 C++ 编程打下坚实基础。



# make_shared和make_unique
在C++中，`std::make_shared` 和 `std::make_unique` 是用于创建智能指针的工厂函数，分别用于 `std::shared_ptr` 和 `std::unique_ptr`。它们提供了更安全、更高效的资源管理方式。以下是详细说明：

---

### **1. `std::make_unique`（C++14 起支持）**
#### **作用**
- 用于创建 `std::unique_ptr`，表示独占所有权的智能指针。
- 避免显式使用 `new` 和 `delete`，减少内存泄漏风险。

#### **语法**
```cpp
auto ptr = std::make_unique<T>(args...);
```
- `T`：对象类型。
- `args...`：传递给 `T` 构造函数的参数。

#### **优势**
1. **异常安全**  
   在复杂表达式中，直接使用 `new` 可能导致内存泄漏（如参数求值顺序问题），而 `make_unique` 保证了原子性。
   ```cpp
   // 潜在的内存泄漏风险（若 process() 抛出异常）
   process(std::unique_ptr<Foo>(new Foo), std::unique_ptr<Bar>(new Bar));

   // 使用 make_unique 保证安全
   process(std::make_unique<Foo>(), std::make_unique<Bar>());
   ```

2. **代码简洁性**  
   避免显式 `new`，代码更简洁。

3. **自动类型推导**  
   `auto` 关键字简化了类型声明。

#### **局限性**
- 不支持自定义删除器。需要自定义删除器时，需直接构造 `std::unique_ptr`。

---

### **2. `std::make_shared`（C++11 起支持）**
#### **作用**
- 用于创建 `std::shared_ptr`，表示共享所有权的智能指针。
- 通过引用计数管理资源，所有 `shared_ptr` 销毁后释放内存。

#### **语法**
```cpp
auto ptr = std::make_shared<T>(args...);
```

#### **优势**
1. **性能优化**  
   将对象内存和控制块（引用计数等）合并为单次内存分配，减少开销。
   ```cpp
   // 两次内存分配：对象 + 控制块
   std::shared_ptr<Foo> p(new Foo);

   // 一次内存分配：对象和控制块合并
   auto p = std::make_shared<Foo>();
   ```

2. **异常安全**  
   类似 `make_unique`，避免因参数求值顺序导致的内存泄漏。

3. **缓存友好性**  
   对象和控制块内存相邻，可能提高缓存局部性。

#### **局限性**
1. **延迟释放内存**  
   若存在 `weak_ptr`，对象内存可能延迟到所有 `weak_ptr` 销毁后才释放（因控制块与对象内存共享）。

2. **不支持自定义分配器/删除器**  
   需要自定义时，需直接构造 `std::shared_ptr`。

3. **构造函数访问限制**  
   若类的构造函数为私有或受保护，需声明 `make_shared` 为友元。

---

### **3. 使用场景对比**
| **场景**               | **`make_unique`** | **make_shared`** | **直接构造**               |
|------------------------|-------------------|------------------|---------------------------|
| 需要独占所有权         | ✔️                | ❌               | `unique_ptr<T>(new T)`    |
| 需要共享所有权         | ❌                | ✔️               | `shared_ptr<T>(new T)`    |
| 需自定义删除器/分配器  | ❌                | ❌               | ✔️                        |
| 性能敏感（减少分配次数）| ❌                | ✔️               | ❌                        |
| 避免控制块内存延迟释放 | ❌                | ❌               | ✔️（直接构造 `shared_ptr`）|

---

### **4. 代码示例**
#### **`make_unique`**
```cpp
#include <memory>

class MyClass {
public:
    MyClass(int a, double b) {}
};

int main() {
    auto ptr = std::make_unique<MyClass>(42, 3.14);
    // ptr 独占 MyClass 对象的所有权
    return 0;
}
```

#### **`make_shared`**
```cpp
#include <memory>

class MyClass {
public:
    MyClass(int a, double b) {}
};

int main() {
    auto ptr = std::make_shared<MyClass>(42, 3.14);
    // ptr 共享 MyClass 对象的所有权
    return 0;
}
```

---

### **5. 总结**
- **优先使用 `make_shared` 和 `make_unique`**：提高代码安全性、可读性和性能。
- **例外情况**：需要自定义删除器/分配器，或明确分离对象与控制块内存时，直接构造智能指针。

通过合理选择工厂函数，可以显著提升C++程序的资源管理效率和健壮性。

---
---
---


# make_shared的补充
你的理解是正确的。`std::make_shared<T>` **一定会构造一个新的 `T` 对象**，而不是直接指向已有的对象。这是由智能指针的所有权语义和内存管理机制决定的。以下是详细的解释：

---

### 1. **`std::make_shared` 的底层行为**
`std::make_shared<T>` 的典型实现步骤：
1. **分配一块连续内存**：同时容纳 `T` 对象和控制块（用于引用计数等元数据）。
2. **在分配的内存中构造 `T` 对象**：通过调用 `T` 的构造函数（拷贝构造或移动构造）。
3. **返回 `std::shared_ptr<T>`**：指向新构造的 `T` 对象。

因此，无论传递的是左值还是右值，`std::make_shared<T>` **始终会创建一个新的 `T` 对象**。

---

### 2. **为什么不能直接指向原来的 `T` 对象？**
#### 问题核心：**所有权与生命周期管理**
- **`std::shared_ptr` 的设计目标**：  
  确保其管理的对象在所有引用消失时被自动销毁。如果直接指向外部对象（如栈上的局部变量或临时对象），可能导致以下问题：
  - **悬空指针**：如果原对象被销毁（例如离开作用域），但 `shared_ptr` 仍指向它，访问时会导致未定义行为。
  - **所有权混乱**：无法保证原对象的生命周期与 `shared_ptr` 的引用计数同步。

#### 示例场景
假设直接让 `shared_ptr` 指向外部对象：
```cpp
void dangerous_push(const T& external_obj) {
    // 错误：直接让 shared_ptr 指向外部对象的地址
    auto data = std::shared_ptr<T>(&external_obj, [](T*){}); // 自定义空删除器（危险！）
    data_queue.push(data);
}

// 调用代码
{
    T local_obj;
    queue.dangerous_push(local_obj); 
} // local_obj 被销毁，但队列中的 shared_ptr 仍指向它 → 悬空指针！
```

---

### 3. **如何避免悬空指针？**
#### 唯一安全的方式：**让 `shared_ptr` 管理自己的对象**
- **拷贝构造**（针对左值参数）：  
  ```cpp
  void push(const T& new_value) {
      // 拷贝构造一个新对象，由 shared_ptr 管理
      auto data = std::make_shared<T>(new_value); 
      data_queue.push(data);
  }
  ```
  - 即使原对象被销毁，队列中的 `shared_ptr` 仍持有独立副本。

- **移动构造**（针对右值参数）：  
  ```cpp
  void push(T&& new_value) {
      // 移动构造一个新对象，避免拷贝
      auto data = std::make_shared<T>(std::move(new_value)); 
      data_queue.push(data);
  }
  ```
  - 原对象（右值）的资源被“窃取”，新对象独立存在。

---

### 4. **性能优化与权衡**
#### 为什么需要拷贝或移动？
- **线程安全**：队列的消费者线程可能在不同线程中修改对象。如果直接共享原对象，需要额外同步机制（如互斥锁），这会增加复杂性。
- **数据隔离**：队列中的对象应该是独立的，避免消费者修改数据时影响其他线程或外部逻辑。

#### 优化手段
- **移动语义**：对于右值参数，优先使用移动构造（零拷贝）。
- **完美转发**：可以通过模板和 `std::forward` 进一步优化参数传递（C++11 起支持）：
  ```cpp
  template<typename U>
  void push(U&& new_value) {
      auto data = std::make_shared<T>(std::forward<U>(new_value));
      std::lock_guard<std::mutex> lk(mut);
      data_queue.push(data);
      data_cond.notify_one();
  }
  ```
  - 统一处理左值和右值，自动选择拷贝或移动。

---

### 5. **总结**
- **`std::make_shared` 必须构造新对象**：这是为了确保 `shared_ptr` 对资源的独占所有权。
- **直接指向原对象是危险的**：会导致悬空指针和所有权混乱。
- **拷贝或移动是必要的代价**：换取线程安全和数据隔离。