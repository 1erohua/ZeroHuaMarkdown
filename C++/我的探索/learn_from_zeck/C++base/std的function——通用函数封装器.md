`std::function` 是 C++ 标准库中的一个通用函数封装器，定义在 `<functional>` 头文件中。它可以存储、复制和调用任何可调用对象（如函数、lambda 表达式、函数对象、绑定表达式等）。`std::function` 提供了一种类型安全的方式来处理各种可调用对象，使得它们可以在统一的接口下使用。

### 1. 基本用法

`std::function` 的模板参数指定了它所封装的可调用对象的签名。例如，`std::function<int(int, int)>` 表示一个接受两个 `int` 参数并返回 `int` 的可调用对象。

#### 示例 1: 封装普通函数

```cpp
#include <iostream>
#include <functional>

int add(int a, int b) {
    return a + b;
}

int main() {
    std::function<int(int, int)> func = add;
    std::cout << "Result: " << func(2, 3) << std::endl;  // 输出: Result: 5
    return 0;
}
```

在这个例子中，`std::function<int(int, int)>` 封装了一个普通的函数 `add`，然后通过 `func(2, 3)` 调用它。

#### 示例 2: 封装 Lambda 表达式

```cpp
#include <iostream>
#include <functional>

int main() {
    std::function<int(int, int)> func = [](int a, int b) {
        return a * b;
    };
    std::cout << "Result: " << func(2, 3) << std::endl;  // 输出: Result: 6
    return 0;
}
```

在这个例子中，`std::function` 封装了一个 lambda 表达式，该表达式接受两个 `int` 参数并返回它们的乘积。

### 2. 封装成员函数

`std::function` 也可以封装成员函数，但需要使用 `std::bind` 或者 lambda 表达式来绑定对象实例。

#### 示例 3: 封装成员函数

```cpp
#include <iostream>
#include <functional>

class MyClass {
public:
    int multiply(int a, int b) {
        return a * b;
    }
};

int main() {
    MyClass obj;
    std::function<int(int, int)> func = std::bind(&MyClass::multiply, &obj, std::placeholders::_1, std::placeholders::_2);
    std::cout << "Result: " << func(2, 3) << std::endl;  // 输出: Result: 6
    return 0;
}
```

在这个例子中，`std::bind` 用于将成员函数 `MyClass::multiply` 绑定到对象 `obj` 上，并将结果存储在 `std::function` 中。

### 3. 封装函数对象

`std::function` 也可以封装函数对象（即重载了 `operator()` 的类实例）。

#### 示例 4: 封装函数对象

```cpp
#include <iostream>
#include <functional>

struct Multiply {
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    Multiply mult;
    std::function<int(int, int)> func = mult;
    std::cout << "Result: " << func(2, 3) << std::endl;  // 输出: Result: 6
    return 0;
}
```

在这个例子中，`std::function` 封装了一个函数对象 `Multiply` 的实例。

### 4. 检查 `std::function` 是否为空

`std::function` 可以处于空的状态（即不包含任何可调用对象）。可以使用 `operator bool()` 或 `operator!` 来检查 `std::function` 是否为空。

#### 示例 5: 检查 `std::function` 是否为空

```cpp
#include <iostream>
#include <functional>

int main() {
    std::function<int(int, int)> func;
    if (!func) {
        std::cout << "func is empty" << std::endl;
    }
    func = [](int a, int b) { return a + b; };
    if (func) {
        std::cout << "func is not empty" << std::endl;
    }
    return 0;
}
```

### 5. `std::function` 的性能考虑

`std::function` 是一个类型擦除的封装器，它在运行时可能会引入一些额外的开销。对于性能敏感的场景，直接使用函数指针或 lambda 表达式可能会更高效。

### 6. `std::function` 的常见用途

- **回调机制**：`std::function` 常用于实现回调机制，允许用户传递自定义的函数或 lambda 表达式作为回调。
- **策略模式**：在策略模式中，`std::function` 可以用来封装不同的策略实现。
- **事件处理**：在事件驱动的系统中，`std::function` 可以用来封装事件处理函数。

### 7. 总结

`std::function` 是 C++ 中一个非常强大的工具，它提供了一种统一的方式来处理各种可调用对象。通过 `std::function`，你可以将函数、lambda 表达式、成员函数、函数对象等封装到一个统一的接口中，并在需要时调用它们。尽管它可能会引入一些运行时开销，但在大多数情况下，这种开销是可以接受的。


#  zeck的使用案例（必看）

`std::function` 是C++11提供的一个通用的可调用包装器，能够封装任何可调用对象，包括普通函数、Lambda表达式、函数对象以及绑定表达式。它实现了类型擦除，使得不同类型的可调用对象可以通过统一的接口进行操作。

### 定义与使用

```cpp
#include <iostream>
#include <functional>

// 普通函数
int add(int a, int b) {
    return a + b;
}

// 函数对象
struct Multiply {
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    // 封装普通函数
    std::function<int(int, int)> func1 = add;
    std::cout << "Add: " << func1(3, 4) << std::endl; // 输出: Add: 7

    // 封装Lambda表达式
    std::function<int(int, int)> func2 = [](int a, int b) -> int {
        return a - b;
    };
    std::cout << "Subtract: " << func2(10, 4) << std::endl; // 输出: Subtract: 6

    // 封装函数对象
    Multiply multiply;
    std::function<int(int, int)> func3 = multiply;
    std::cout << "Multiply: " << func3(3, 4) << std::endl; // 输出: Multiply: 12

    return 0;
}
```

### 特点

- **类型擦除：** 可以存储任何符合签名的可调用对象。
- **灵活性：** 支持动态改变存储的可调用对象。
- **性能开销：** 相比于直接使用函数指针或Lambda，`std::function` 可能带来一定的性能开销，尤其是在频繁调用时。

### 用法场景

- **回调函数的传递。**
- **事件处理系统。**
- **策略模式的实现。**

### 示例：回调机制

```cpp
#include <iostream>
#include <functional>

// 定义回调类型
using Callback = std::function<void(int)>;

// 触发事件的函数
void triggerEvent(Callback cb, int value) {
    // 事件发生，调用回调
    cb(value);
}

int main() {
    // 使用Lambda作为回调
    triggerEvent([](int x) {
        std::cout << "事件触发，值为: " << x << std::endl;
    }, 42); // 输出: 事件触发，值为: 42

    // 使用仿函数作为回调
    struct Printer {
        void operator()(int x) const {
            std::cout << "Printer打印值: " << x << std::endl;
        }
    } printer;

    triggerEvent(printer, 100); // 输出: Printer打印值: 100

    return 0;
}
```

### 存储和调用不同类型的可调用对象

`std::function` 可以在容器中存储各种不同类型的可调用对象，**只要它们符合指定的签名。**

```cpp
#include <iostream>
#include <functional>
#include <vector>

int add(int a, int b) {
    return a + b;
}

struct Multiply {
    int operator()(int a, int b) const {
        return a * b;
    }
};

int main() {
    std::vector<std::function<int(int, int)>> operations;

    // 添加不同类型的可调用对象
    // 牛逼，用数组存储批处理
    operations.emplace_back(add); // 普通函数
    operations.emplace_back(Multiply()); // 仿函数
    operations.emplace_back([](int a, int b) -> int { return a - b; }); // Lambda

    // 执行所有操作
    for(auto& op : operations) {
        std::cout << op(10, 5) << " "; // 输出: 15 50 5
    }
    std::cout << std::endl;

    return 0;
}
```