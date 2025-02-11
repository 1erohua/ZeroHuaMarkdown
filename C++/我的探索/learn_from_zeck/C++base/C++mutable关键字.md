`mutable` 是 C++ 中的一个关键字，用于修饰类的成员变量，允许在 `const` 成员函数中修改这些变量。通常情况下，`const` 成员函数不能修改类的任何成员变量，但使用 `mutable` 修饰的变量可以在 `const` 成员函数中被修改。

### 使用场景
`mutable` 主要用于以下场景：
1. **缓存计算结果**：在 `const` 成员函数中缓存某些计算结果，避免重复计算。
2. **线程安全**：在多线程环境中，使用 `mutable` 修饰互斥锁或原子变量，以便在 `const` 成员函数中修改它们。
3. **调试和日志**：在 `const` 成员函数中记录调试信息或日志。

### 示例代码
```cpp
#include <iostream>
#include <mutex>

class Example {
public:
    int getValue() const {
        // 模拟缓存计算结果
        if (!cacheValid) {
            // 由于 cachedValue 是 mutable，可以在 const 函数中修改
            cachedValue = computeExpensiveValue();
            cacheValid = true;
        }
        return cachedValue;
    }

private:
    mutable int cachedValue;  // 缓存的值
    mutable bool cacheValid = false;  // 缓存是否有效

    int computeExpensiveValue() const {
        // 模拟一个耗时的计算
        return 42;
    }
};

class ThreadSafeExample {
public:
    void doSomething() const {
        std::lock_guard<std::mutex> lock(mutex);  // 修改 mutex，需要 mutable
        // 线程安全的操作
    }

private:
    mutable std::mutex mutex;  // 互斥锁，用于线程安全
};

int main() {
    Example example;
    std::cout << "Cached value: " << example.getValue() << std::endl;

    ThreadSafeExample tsExample;
    tsExample.doSomething();

    return 0;
}
```

### 关键点
1. **`mutable` 修饰的变量可以在 `const` 成员函数中被修改**。
2. **`mutable` 通常用于与对象逻辑状态无关的变量**，例如缓存、调试信息或线程同步工具。
3. **滥用 `mutable` 可能导致代码难以理解和维护**，因此应谨慎使用。

### 总结
`mutable` 提供了一种在 `const` 成员函数中修改特定成员变量的机制，适用于缓存、线程安全等场景。合理使用 `mutable` 可以增强代码的灵活性和性能，但需注意避免滥用。