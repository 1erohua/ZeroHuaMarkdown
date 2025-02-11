为什么要这么做？
在C++中，防止对象被复制的原因主要有以下几点：

### 1. **资源管理**
   如果对象管理着独占资源（如文件句柄、网络连接、动态内存等），复制可能导致多个对象共享同一资源，引发资源释放问题（如双重释放）。为防止这种情况，可以禁用复制。

   ```cpp
   class ResourceHolder {
   public:
       ResourceHolder() { /* 获取资源 */ }
       ~ResourceHolder() { /* 释放资源 */ }
       ResourceHolder(const ResourceHolder&) = delete; // 禁用复制构造函数
       ResourceHolder& operator=(const ResourceHolder&) = delete; // 禁用复制赋值运算符
   };
   ```

### 2. **唯一性**
   某些对象在系统中应是唯一的（如单例类），复制会破坏这种唯一性。通过禁用复制，可以确保对象的唯一性。

   ```cpp
   class Singleton {
   public:
       static Singleton& getInstance() {
           static Singleton instance;
           return instance;
       }
       Singleton(const Singleton&) = delete; // 禁用复制构造函数
       Singleton& operator=(const Singleton&) = delete; // 禁用复制赋值运算符
   private:
       Singleton() {} // 私有构造函数
   };
   ```

### 3. **性能优化**
   复制某些对象可能代价高昂（如包含大量数据或复杂状态的对象），禁用复制可以避免不必要的性能损耗。

   ```cpp
   class LargeData {
   public:
       LargeData() { /* 初始化大量数据 */ }
       LargeData(const LargeData&) = delete; // 禁用复制构造函数
       LargeData& operator=(const LargeData&) = delete; // 禁用复制赋值运算符
   };
   ```

### 4. **逻辑错误**
   某些对象的复制在逻辑上没有意义（如表示线程或互斥锁的对象），复制可能导致逻辑错误或未定义行为。

   ```cpp
   class Mutex {
   public:
       Mutex() { /* 初始化互斥锁 */ }
       ~Mutex() { /* 销毁互斥锁 */ }
       Mutex(const Mutex&) = delete; // 禁用复制构造函数
       Mutex& operator=(const Mutex&) = delete; // 禁用复制赋值运算符
   };
   ```

### 5. **避免浅拷贝问题**
   如果对象包含指针或动态分配的资源，默认的浅拷贝会导致多个对象共享同一资源，可能引发问题。禁用复制可以避免浅拷贝问题。

   ```cpp
   class ShallowCopyExample {
   public:
       int* data;
       ShallowCopyExample() { data = new int(10); }
       ~ShallowCopyExample() { delete data; }
       ShallowCopyExample(const ShallowCopyExample&) = delete; // 禁用复制构造函数
       ShallowCopyExample& operator=(const ShallowCopyExample&) = delete; // 禁用复制赋值运算符
   };
   ```

### 总结
防止对象被复制主要是为了确保资源管理、唯一性、性能优化、逻辑正确性以及避免浅拷贝问题。通过禁用复制构造函数和复制赋值运算符，可以有效避免这些问题。

怎么做

在C++中，可以通过以下几种方式防止对象被复制：

### 1. 删除拷贝构造函数和拷贝赋值运算符
C++11 引入了 `= delete` 语法，可以显式删除拷贝构造函数和拷贝赋值运算符，从而防止对象被复制。

```cpp
class NonCopyable {
public:
    NonCopyable() = default;

    // 删除拷贝构造函数
    NonCopyable(const NonCopyable&) = delete;

    // 删除拷贝赋值运算符
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

### 2. 将拷贝构造函数和拷贝赋值运算符声明为 `private`
在 C++11 之前，通常通过将拷贝构造函数和拷贝赋值运算符声明为 `private` 并**不提供实现**来防止对象被复制。

```cpp
class NonCopyable {
private:
    // 声明为 private 并不提供实现
    NonCopyable(const NonCopyable&);
    NonCopyable& operator=(const NonCopyable&);

public:
    NonCopyable() = default;
};
```

### 3. 继承 `boost::noncopyable`
如果你使用 Boost 库，可以直接继承 `boost::noncopyable` 类，它会自动禁用拷贝构造函数和拷贝赋值运算符。

```cpp
#include <boost/noncopyable.hpp>

class NonCopyable : private boost::noncopyable {
public:
    NonCopyable() = default;
};
```

### 4. 使用 `= delete` 删除移动构造函数和移动赋值运算符
如果你还想防止对象被移动，可以删除移动构造函数和移动赋值运算符。

```cpp
class NonCopyable {
public:
    NonCopyable() = default;

    // 删除拷贝构造函数
    NonCopyable(const NonCopyable&) = delete;

    // 删除拷贝赋值运算符
    NonCopyable& operator=(const NonCopyable&) = delete;

    // 删除移动构造函数
    NonCopyable(NonCopyable&&) = delete;

    // 删除移动赋值运算符
    NonCopyable& operator=(NonCopyable&&) = delete;
};
```

### 总结
- **C++11 及以后**：推荐使用 `= delete` 删除拷贝构造函数和拷贝赋值运算符。
- **C++11 之前**：将拷贝构造函数和拷贝赋值运算符声明为 `private` 并不提供实现。
- **使用 Boost 库**：可以继承 `boost::noncopyable` 来简化代码。

通过这些方法，你可以有效地防止对象被复制。
