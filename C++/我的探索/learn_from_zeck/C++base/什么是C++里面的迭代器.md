## 迭代器分类

### 1. 迭代器（Iterator）简介

在 C++ 中，**迭代器** 是一种用于遍历容器（如 `std::vector`、`std::list` 等）元素的对象。它们提供了类似指针的接口，使得算法可以独立于具体的容器而工作。迭代器的设计允许算法以统一的方式处理不同类型的容器。

---

### 2. 迭代器类别（Iterator Categories）

为了使不同类型的迭代器能够支持不同的操作，C++ 标准库将迭代器分为以下几种类别，每种类别支持的操作能力逐级增强：

1. **输入迭代器（Input Iterator）**
2. **输出迭代器（Output Iterator）**
3. **前向迭代器（Forward Iterator）**
4. **双向迭代器（Bidirectional Iterator）**
5. **随机访问迭代器（Random Access Iterator）**
6. **无效迭代器（Contiguous Iterator）**（C++20 引入）

每个类别都继承自前一个类别，具备更强的功能。例如，双向迭代器不仅支持前向迭代器的所有操作，还支持反向迭代（即可以向后移动）。

**主要迭代器类别及其特性**

|类别|支持的操作|示例容器|
|---|---|---|
|输入迭代器|只读访问、单向前进|单向链表 `std::forward_list`|
|输出迭代器|只写访问、单向前进|输出流 `std::ostream_iterator`|
|前向迭代器|读写访问、单向前进|向量 `std::vector`|
|双向迭代器|读写访问、单向前进和反向迭代|双向链表 `std::list`|
|随机访问迭代器|读写访问、单向前进、反向迭代、跳跃移动（支持算术运算）|向量 `std::vector`、队列 `std::deque`|
|无效迭代器（新）|随机访问迭代器的所有功能，且元素在内存中连续排列|新的 C++ 容器如 `std::span`|

---

### 3. `iterator_category` 的作用

`iterator_category` 是迭代器类型中的一个别名，用于标识该迭代器所属的类别。它是标准库中 **迭代器特性（Iterator Traits）** 的一部分，标准算法会根据迭代器的类别优化其行为。

### 为什么需要 `iterator_category`

标准库中的算法（如 `std::sort`、`std::find` 等）需要知道迭代器支持哪些操作，以便选择最优的实现方式。例如：

- 对于**随机访问迭代器**，可以使用快速的随机访问算法（如快速排序）。
- 对于**双向迭代器**，只能使用适用于双向迭代的算法（如归并排序）。
- 对于**输入迭代器**，只能进行单次遍历，许多复杂算法无法使用。

通过指定 `iterator_category`，你可以让标准算法了解你自定义迭代器的能力，从而选择合适的方法进行操作。

### `iterator_category` 的声明

在你的自定义迭代器类中，通过以下方式声明迭代器类别：

```cpp
using iterator_category = std::bidirectional_iterator_tag;
```

这表示该迭代器是一个 **双向迭代器**，支持向前和向后遍历。

---

### 4. `std::bidirectional_iterator_tag` 详解

`std::bidirectional_iterator_tag` 是一个标签（Tag），用于标识迭代器类别。C++ 标准库中有多个这样的标签，分别对应不同的迭代器类别：

- `std::input_iterator_tag`
- `std::output_iterator_tag`
- `std::forward_iterator_tag`
- `std::bidirectional_iterator_tag`
- `std::random_access_iterator_tag`
- `std::contiguous_iterator_tag`（C++20）

这些标签本质上是空的结构体，用于类型区分。在标准算法中，通常会通过这些标签进行 **重载选择（Overload Resolution）** 或 **特化（Specialization）**，以实现针对不同迭代器类别的优化。

### 继承关系

迭代器标签是有继承关系的：

- `std::forward_iterator_tag` 继承自 `std::input_iterator_tag`
- `std::bidirectional_iterator_tag` 继承自 `std::forward_iterator_tag`
- `std::random_access_iterator_tag` 继承自 `std::bidirectional_iterator_tag`
- `std::contiguous_iterator_tag` 继承自 `std::random_access_iterator_tag`

这种继承关系反映了迭代器类别的能力层级。例如，**双向迭代器** 具备 **前向迭代器** 的所有能力，加上反向遍历的能力。

---

### 5. 迭代器特性（Iterator Traits）详解

C++ 提供了 **迭代器特性（Iterator Traits）**，通过模板类 `std::iterator_traits` 来获取迭代器的相关信息。通过这些特性，标准算法可以泛化地处理不同类型的迭代器。

### 迭代器特性包含的信息

`std::iterator_traits` 提供以下信息：

- `iterator_category`：迭代器类别标签。
- `value_type`：迭代器指向的元素类型。
- `difference_type`：迭代器间的距离类型（通常是 `std::ptrdiff_t`）。
- `pointer`：指向元素的指针类型。
- `reference`：对元素的引用类型。

### 自定义迭代器与 `iterator_traits`

当你定义自己的迭代器时，确保提供这些类型别名，以便标准库算法能够正确识别和使用你的迭代器。例如：

```cpp
template<typename T>
class Iterator {
public:
    using iterator_category = std::bidirectional_iterator_tag;
    using value_type        = T;
    using difference_type   = std::ptrdiff_t;
    using pointer           = T*;
    using reference         = T&;

    // 其他成员函数...
};
```

这样，使用 `std::iterator_traits<Iterator<T>>` 时，就能正确获取迭代器的特性。