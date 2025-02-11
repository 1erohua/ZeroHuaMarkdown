再回顾——链接：[[七、C++ 并发三剑客#公司级线程池？]]

直到现在才发现`std::bind`的牛逼，能将一个有参数传递的函数自动打包成**无参数传递的参数**
请看上面链接内，commit函数的这一段：
```cpp
        auto task = std::make_shared<std::packaged_task<ReturnType()>>
        (
            // std::forward返回f与args的完美转发 形态的f和args
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

```
正是因为`std::bind`将它们打包成了无参数函数，这样packaged_task所存放的函数类型就是
`ReturnType()`

---

`std::bind` 是 C++ 标准库中的一个工具，用于将函数或成员函数与其参数绑定，生成一个可调用对象（通常是一个函数对象）。它可以部分应用函数参数，或者重新排列参数顺序，从而创建一个新的可调用对象。`std::bind` 在 C++11 中引入，通常与 `<functional>` 头文件一起使用。

### `std::bind` 的基本用法

`std::bind` 的基本语法如下：

```cpp
#include <functional>

auto bound_callable = std::bind(callable, arg1, arg2, ...);
```

- `callable` 可以是函数、函数指针、成员函数指针、函数对象等。
- `arg1, arg2, ...` 是传递给 `callable` 的参数。这些参数可以是具体的值、占位符（`std::placeholders::_1`, `std::placeholders::_2`, ...）或其他可调用对象。

#### 1. 绑定普通函数

```cpp
#include <iostream>
#include <functional>

void print_sum(int a, int b) {
    std::cout << a + b << std::endl;
}

int main() {
    auto bound_print = std::bind(print_sum, 10, 20);
    bound_print();  // 输出 30
    return 0;
}
```

在这个例子中，`std::bind` 将 `print_sum` 函数与参数 `10` 和 `20` 绑定，生成了一个可调用对象 `bound_print`。调用 `bound_print()` 时，它会执行 `print_sum(10, 20)`。

#### 2. 使用占位符

`std::placeholders::_1`, `std::placeholders::_2`, ... 是占位符，表示在调用时传递的参数。

```cpp
#include <iostream>
#include <functional>

void print_sum(int a, int b) {
    std::cout << a + b << std::endl;
}

int main() {
    auto bound_print = std::bind(print_sum, std::placeholders::_1, 20);
    bound_print(10);  // 输出 30
    return 0;
}
```

在这个例子中，`std::placeholders::_1` 表示第一个参数将在调用 `bound_print` 时传递。调用 `bound_print(10)` 时，它会执行 `print_sum(10, 20)`。

#### 3. 绑定成员函数

绑定成员函数时，需要传递对象的指针或引用作为第一个参数。

```cpp
#include <iostream>
#include <functional>

struct MyClass {
    void print_sum(int a, int b) {
        std::cout << a + b << std::endl;
    }
};

int main() {
    MyClass obj;
    auto bound_print = std::bind(&MyClass::print_sum, &obj, std::placeholders::_1, 20);
    bound_print(10);  // 输出 30
    return 0;
}
```

在这个例子中，`std::bind` 绑定了 `MyClass` 的成员函数 `print_sum`，并传递了 `obj` 的指针作为第一个参数。调用 `bound_print(10)` 时，它会执行 `obj.print_sum(10, 20)`。

#### 4. 重新排列参数顺序

`std::bind` 还可以用来重新排列参数的顺序。

```cpp
#include <iostream>
#include <functional>

void print_diff(int a, int b) {
    std::cout << a - b << std::endl;
}

int main() {
    auto bound_print = std::bind(print_diff, std::placeholders::_2, std::placeholders::_1);
    bound_print(10, 20);  // 输出 10 (20 - 10)
    return 0;
}
```

在这个例子中，`std::bind` 将 `print_diff` 的参数顺序重新排列。调用 `bound_print(10, 20)` 时，它会执行 `print_diff(20, 10)`。

### `std::bind` 与 Lambda 表达式的对比

`std::bind` 和 Lambda 表达式都可以用来创建可调用对象，但它们的使用场景和语法有所不同。

#### 1. 语法简洁性

Lambda 表达式通常比 `std::bind` 更简洁，尤其是在处理简单的函数绑定时。

```cpp
// 使用 std::bind
auto bound_print = std::bind(print_sum, std::placeholders::_1, 20);

// 使用 Lambda 表达式
auto lambda_print = [](int a) { print_sum(a, 20); };
```

在这个例子中，Lambda 表达式更加直观和简洁。

#### 2. 灵活性

Lambda 表达式在处理复杂逻辑时更加灵活，因为它可以在函数体内直接编写代码，而不需要像 `std::bind` 那样通过占位符来传递参数。

```cpp
// 使用 std::bind
auto bound_print = std::bind(print_sum, std::placeholders::_1, std::placeholders::_2);

// 使用 Lambda 表达式
auto lambda_print = [](int a, int b) { print_sum(a, b); };
```

#### 3. 性能

在某些情况下，Lambda 表达式可能比 `std::bind` 更高效，因为 Lambda 表达式通常会被编译器内联优化，而 `std::bind` 可能会引入额外的函数调用开销。

#### 4. 可读性

Lambda 表达式通常更具可读性，尤其是在处理复杂逻辑时。`std::bind` 的语法可能会让代码变得难以理解，尤其是在使用多个占位符时。

```cpp
// 使用 std::bind
auto bound_print = std::bind(print_sum, std::placeholders::_2, std::placeholders::_1);

// 使用 Lambda 表达式
auto lambda_print = [](int a, int b) { print_sum(b, a); };
```

在这个例子中，Lambda 表达式更直观地表达了参数的顺序。

### 总结

- `std::bind` 是一个强大的工具，可以用于部分应用函数参数、重新排列参数顺序等。它在处理简单的函数绑定时非常有用，但在处理复杂逻辑时可能会显得笨拙。
- Lambda 表达式通常更简洁、灵活和高效，尤其是在处理复杂逻辑时。它在现代 C++ 中更受欢迎，通常被认为是 `std::bind` 的替代品。

在实际开发中，建议优先使用 Lambda 表达式，除非有特定的需求需要使用 `std::bind`。