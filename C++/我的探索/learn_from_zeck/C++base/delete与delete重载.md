在C++中，`delete` 是一个用于释放动态分配内存的操作符。它通常与 `new` 操作符一起使用，`new` 用于在堆上分配内存，而 `delete` 用于释放这些内存，以防止内存泄漏。
> delete会调用析构函数，这是最重要的！！
### 1. 基本用法

当你使用 `new` 操作符动态分配内存时，例如：

```cpp
int* ptr = new int;
```

你需要在不再需要这块内存时使用 `delete` 来释放它：

```cpp
delete ptr;
```

### 2. 释放数组内存

如果你使用 `new[]` 分配了一个数组，例如：

```cpp
int* arr = new int[10];
```

你需要使用 `delete[]` 来释放数组内存：

```cpp
delete[] arr;
```

注意：使用 `delete` 释放数组内存或使用 `delete[]` 释放单个对象的内存都是未定义行为，可能导致程序崩溃或其他不可预见的错误。

### 3. `delete` 的作用

- **释放内存**：`delete` 操作符会释放指针所指向的内存，使得这块内存可以被重新分配。
- **调用析构函数**：如果指针指向的是一个对象，`delete` 会调用该对象的析构函数，以确保对象内部的资源被正确释放。

### 4. `delete` 与 `nullptr`

在释放内存后，通常建议将指针设置为 `nullptr`，以避免悬空指针（dangling pointer）问题：

```cpp
delete ptr;
ptr = nullptr;
```

这样做可以防止在后续代码中误用已经释放的内存。

### 5. 注意事项

- **不要重复释放内存**：对同一个指针多次调用 `delete` 是未定义行为。
- **不要释放栈上的内存**：`delete` 只能用于释放通过 `new` 分配的内存，不能用于释放栈上的内存。
- **不要忘记释放内存**：忘记释放动态分配的内存会导致内存泄漏。

### 6. 示例代码

```cpp
#include <iostream>

class MyClass {
public:
    MyClass() { std::cout << "Constructor called!" << std::endl; }
    ~MyClass() { std::cout << "Destructor called!" << std::endl; }
};

int main() {
    // 分配单个对象
    MyClass* obj = new MyClass;
    delete obj;
    obj = nullptr;

    // 分配数组
    int* arr = new int[10];
    delete[] arr;
    arr = nullptr;

    return 0;
}
```

在这个示例中，`MyClass` 的构造函数和析构函数分别在对象创建和销毁时被调用。`delete` 操作符确保了对象的内存被正确释放，并且析构函数被调用。

### 7. 智能指针

为了避免手动管理内存的复杂性，C++11 引入了智能指针（如 `std::unique_ptr` 和 `std::shared_ptr`），它们可以自动管理内存的释放，减少内存泄漏的风险。

```cpp
#include <memory>

int main() {
    std::unique_ptr<MyClass> obj = std::make_unique<MyClass>();
    // 不需要手动调用 delete，智能指针会自动释放内存
    return 0;
}
```

智能指针在超出作用域时会自动释放所管理的内存，从而简化了内存管理。

### 总结

`delete` 是 C++ 中用于释放动态分配内存的重要操作符。正确使用 `delete` 和 `delete[]` 可以避免内存泄漏和其他内存相关的问题。然而，现代 C++ 更推荐使用智能指针来自动管理内存，以减少手动管理内存带来的风险。














在C++中，`delete` 是一个操作符，用于释放由 `new` 操作符动态分配的内存。通常情况下，`delete` 操作符的行为是固定的，但C++允许我们通过重载 `delete` 操作符来自定义内存释放的行为。

### 重载 `delete` 操作符

重载 `delete` 操作符通常用于自定义内存管理，例如在特定的内存池中分配和释放内存，或者进行内存泄漏检测等。

#### 基本语法

```cpp
void operator delete(void* ptr) noexcept;
void operator delete[](void* ptr) noexcept;
```

你可以重载全局的 `delete` 操作符，也可以重载类特定的 `delete` 操作符。

#### 示例：重载全局 `delete` 操作符

```cpp
#include <iostream>
#include <cstdlib>

// 重载全局的 delete 操作符
void operator delete(void* ptr) noexcept {
    std::cout << "Custom global delete called." << std::endl;
    std::free(ptr);
}

int main() {
    int* p = new int(10);
    delete p;  // 调用重载的 delete 操作符
    return 0;
}
```

#### 示例：重载类特定的 `delete` 操作符

```cpp
#include <iostream>
#include <cstdlib>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructed." << std::endl;
    }

    ~MyClass() {
        std::cout << "MyClass destroyed." << std::endl;
    }

    // 重载类特定的 delete 操作符
    void operator delete(void* ptr) noexcept {
        std::cout << "Custom delete for MyClass called." << std::endl;
        std::free(ptr);
    }
};

int main() {
    MyClass* obj = new MyClass();
    delete obj;  // 调用重载的 delete 操作符
    return 0;
}
```

#### 重载 `delete[]` 操作符

如果你使用 `new[]` 分配数组，你也需要重载 `delete[]` 操作符。

```cpp
#include <iostream>
#include <cstdlib>

class MyClass {
public:
    MyClass() {
        std::cout << "MyClass constructed." << std::endl;
    }

    ~MyClass() {
        std::cout << "MyClass destroyed." << std::endl;
    }

    // 重载类特定的 delete[] 操作符
    void operator delete[](void* ptr) noexcept {
        std::cout << "Custom delete[] for MyClass called." << std::endl;
        std::free(ptr);
    }
};

int main() {
    MyClass* arr = new MyClass[3];
    delete[] arr;  // 调用重载的 delete[] 操作符
    return 0;
}
```

### 注意事项

1. **异常安全性**：重载的 `delete` 操作符应该标记为 `noexcept`，因为 `delete` 操作符在正常情况下不应该抛出异常。

2. **内存对齐**：如果你在重载的 `delete` 操作符中使用了自定义的内存分配器，确保内存对齐符合要求。

3. **匹配 `new` 和 `delete`**：如果你重载了 `new` 操作符，通常也需要重载对应的 `delete` 操作符，以确保内存管理的一致性。

4. **调试和日志**：重载 `delete` 操作符可以用于调试目的，例如记录内存释放的情况，检测内存泄漏等。

通过重载 `delete` 操作符，你可以更灵活地控制内存的释放过程，适应特定的应用场景。

