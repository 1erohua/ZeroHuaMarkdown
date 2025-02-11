# std::map
`std::map` 和 `std::unordered_map` 是 C++ 标准库中的两种关联容器，用于存储键值对。它们的用法和实现原理有所不同，下面详细讲解它们的用法和实现原理。

### 1. `std::map`

#### 用法
`std::map` 是一个有序的关联容器，基于红黑树（一种自平衡二叉查找树）实现。它存储键值对，并且按键的升序排序。

```cpp
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> myMap;

    // 插入元素
    myMap["apple"] = 5;
    myMap["banana"] = 3;
    myMap["cherry"] = 7;

    // 访问元素
    std::cout << "apple: " << myMap["apple"] << std::endl;

    // 遍历元素
    for (const auto& pair : myMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    // 查找元素
    auto it = myMap.find("banana");
    if (it != myMap.end()) {
        std::cout << "Found banana: " << it->second << std::endl;
    }

    // 删除元素
    myMap.erase("cherry");

    return 0;
}
```

#### 实现原理
`std::map` 基于红黑树实现，红黑树是一种自平衡二叉查找树，具有以下特性：
- 每个节点是红色或黑色。
- 根节点是黑色。
- 每个叶子节点（NIL节点）是黑色。
- 如果一个节点是红色，则它的两个子节点都是黑色。
- 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

这些特性保证了红黑树的平衡，使得插入、删除和查找操作的时间复杂度为 O(log n)。

### 2. `std::unordered_map`

#### 用法
`std::unordered_map` 是一个无序的关联容器，基于哈希表实现。它存储键值对，并且不保证元素的顺序。

```cpp
#include <iostream>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> myUnorderedMap;

    // 插入元素
    myUnorderedMap["apple"] = 5;
    myUnorderedMap["banana"] = 3;
    myUnorderedMap["cherry"] = 7;

    // 访问元素
    std::cout << "apple: " << myUnorderedMap["apple"] << std::endl;

    // 遍历元素
    for (const auto& pair : myUnorderedMap) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    // 查找元素
    auto it = myUnorderedMap.find("banana");
    if (it != myUnorderedMap.end()) {
        std::cout << "Found banana: " << it->second << std::endl;
    }

    // 删除元素
    myUnorderedMap.erase("cherry");

    return 0;
}
```

#### 实现原理
`std::unordered_map` 基于哈希表实现，哈希表通过哈希函数将键映射到桶（bucket）中。哈希表的主要操作包括：
- **插入**：通过哈希函数计算键的哈希值，将键值对插入到对应的桶中。
- **查找**：通过哈希函数计算键的哈希值，查找对应的桶中的键值对。
- **删除**：通过哈希函数计算键的哈希值，删除对应的桶中的键值对。

哈希表的平均时间复杂度为 O(1)，但在最坏情况下（哈希冲突严重时）可能退化为 O(n)。

### 3. 比较

| 特性                | `std::map`                  | `std::unordered_map`       |
|---------------------|----------------------------|----------------------------|
| 实现                | 红黑树                     | 哈希表                     |
| 顺序                | 有序                       | 无序                       |
| 时间复杂度（平均）  | 插入、删除、查找：O(log n) | 插入、删除、查找：O(1)     |
| 时间复杂度（最坏）  | O(log n)                   | O(n)                       |
| 内存使用            | 较低                       | 较高                       |
| 适用场景            | 需要有序存储               | 需要快速查找               |

### 4. 选择建议
- 如果需要有序存储键值对，或者需要按顺序遍历元素，选择 `std::map`。
- 如果需要快速查找、插入和删除操作，并且不关心元素的顺序，选择 `std::unordered_map`。

通过理解它们的用法和实现原理，可以根据具体需求选择合适的容器。


# std::set 

---

在C++中，`set` 和 `unordered_set` 是两种常用的关联容器，用于存储唯一的元素。它们的主要区别在于底层实现和性能特征。下面详细讲解它们的用法和实现原理。

### 1. `std::set`

#### 用法
`std::set` 是一个有序的关联容器，基于红黑树（一种自平衡二叉搜索树）实现。它存储唯一的元素，并且元素按照升序排列（默认情况下）。

- **头文件**: `<set>`
- **常用操作**:
  - `insert(value)`: 插入元素。
  - `erase(value)`: 删除元素。
  - `find(value)`: 查找元素，返回迭代器。
  - `size()`: 返回元素个数。
  - `empty()`: 判断是否为空。
  - `begin()`, `end()`: 返回迭代器。

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s;
    s.insert(3);
    s.insert(1);
    s.insert(4);
    s.insert(1); // 重复元素不会被插入

    for (int x : s) {
        std::cout << x << " "; // 输出: 1 3 4
    }

    if (s.find(3) != s.end()) {
        std::cout << "3 is in the set" << std::endl;
    }

    s.erase(3);
    std::cout << "Size after erase: " << s.size() << std::endl; // 输出: 2
}
```

#### 实现原理
`std::set` 通常基于红黑树实现。红黑树是一种自平衡二叉搜索树，具有以下特性：
- 每个节点是红色或黑色。
- 根节点是黑色。
- 每个叶子节点（NIL节点）是黑色。
- 如果一个节点是红色，则它的两个子节点都是黑色。
- 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

这些特性保证了红黑树在最坏情况下的插入、删除和查找操作的时间复杂度为 O(log n)。

### 2. `std::unordered_set`

#### 用法
`std::unordered_set` 是一个无序的关联容器，基于哈希表实现。它存储唯一的元素，但不保证元素的顺序。

- **头文件**: `<unordered_set>`
- **常用操作**:
  - `insert(value)`: 插入元素。
  - `erase(value)`: 删除元素。
  - `find(value)`: 查找元素，返回迭代器。
  - `size()`: 返回元素个数。
  - `empty()`: 判断是否为空。
  - `begin()`, `end()`: 返回迭代器。

```cpp
#include <iostream>
#include <unordered_set>

int main() {
    std::unordered_set<int> us;
    us.insert(3);
    us.insert(1);
    us.insert(4);
    us.insert(1); // 重复元素不会被插入

    for (int x : us) {
        std::cout << x << " "; // 输出顺序不确定
    }

    if (us.find(3) != us.end()) {
        std::cout << "3 is in the unordered_set" << std::endl;
    }

    us.erase(3);
    std::cout << "Size after erase: " << us.size() << std::endl; // 输出: 2
}
```

#### 实现原理
`std::unordered_set` 基于哈希表实现。哈希表通过哈希函数将元素映射到桶（bucket）中，每个桶可以存储多个元素。理想情况下，哈希表的插入、删除和查找操作的时间复杂度为 O(1)，但在最坏情况下（哈希冲突严重时）可能退化为 O(n)。

- **哈希函数**: 将元素映射到桶的索引。
- **冲突处理**: 通常使用链地址法（每个桶是一个链表）或开放地址法（在冲突时寻找下一个空闲桶）来处理哈希冲突。

### 3. 对比

| 特性                | `std::set`                  | `std::unordered_set`        |
|---------------------|-----------------------------|-----------------------------|
| **底层实现**         | 红黑树                      | 哈希表                      |
| **元素顺序**         | 有序                        | 无序                        |
| **插入/删除/查找**   | O(log n)                    | 平均 O(1)，最坏 O(n)        |
| **内存使用**         | 较低                        | 较高                        |
| **适用场景**         | 需要有序元素或频繁范围查询  | 需要快速查找、插入、删除    |

### 4. 总结
- **`std::set`**: 适合需要有序存储和频繁范围查询的场景。
- **`std::unordered_set`**: 适合需要快速查找、插入和删除的场景，且不关心元素顺序。

根据具体需求选择合适的容器可以提高程序的效率。


# std::stack、queue、priority_queue
在C++中，`stack`、`queue`和`priority_queue`是标准模板库（STL）提供的三种常用容器适配器。它们基于其他容器（如`deque`或`vector`）实现，提供了特定的数据结构和操作。下面详细讲解它们的用法和实现原理。

### 1. `stack`（栈）

#### 用法
`stack` 是一种后进先出（LIFO, Last In First Out）的数据结构。它只允许在栈顶进行插入和删除操作。

- **头文件**: `<stack>`
- **常用操作**:
  - `push()`: 将元素压入栈顶。
  - `pop()`: 弹出栈顶元素。
  - `top()`: 访问栈顶元素。
  - `empty()`: 判断栈是否为空。
  - `size()`: 返回栈中元素的数量。

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> s;

    s.push(10); // 压入10
    s.push(20); // 压入20
    s.push(30); // 压入30

    while (!s.empty()) {
        std::cout << s.top() << " "; // 访问栈顶元素
        s.pop(); // 弹出栈顶元素
    }

    return 0;
}
```

#### 实现原理
`stack` 通常基于 `deque` 或 `vector` 实现。默认情况下，`stack` 使用 `deque` 作为底层容器。

- **底层容器**: `deque`（默认）或 `vector`
- **操作复杂度**:
  - `push()`: O(1)
  - `pop()`: O(1)
  - `top()`: O(1)
  - `empty()`: O(1)
  - `size()`: O(1)

### 2. `queue`（队列）

#### 用法
`queue` 是一种先进先出（FIFO, First In First Out）的数据结构。它允许在队尾插入元素，在队头删除元素。

- **头文件**: `<queue>`
- **常用操作**:
  - `push()`: 将元素插入队尾。
  - `pop()`: 删除队头元素。
  - `front()`: 访问队头元素。
  - `back()`: 访问队尾元素。
  - `empty()`: 判断队列是否为空。
  - `size()`: 返回队列中元素的数量。

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;

    q.push(10); // 插入10
    q.push(20); // 插入20
    q.push(30); // 插入30

    while (!q.empty()) {
        std::cout << q.front() << " "; // 访问队头元素
        q.pop(); // 删除队头元素
    }

    return 0;
}
```

#### 实现原理
`queue` 通常基于 `deque` 实现。默认情况下，`queue` 使用 `deque` 作为底层容器。

- **底层容器**: `deque`（默认）
- **操作复杂度**:
  - `push()`: O(1)
  - `pop()`: O(1)
  - `front()`: O(1)
  - `back()`: O(1)
  - `empty()`: O(1)
  - `size()`: O(1)

C++中的优先队列（`priority_queue`）是一种容器适配器，它基于堆数据结构实现，允许你以特定的顺序处理元素。默认情况下，优先队列是一个最大堆，即队列中的最大元素总是位于队列的顶部。你可以通过自定义比较函数来改变元素的优先级顺序。

## priority_queue
### 1. 基本概念

- **堆**：优先队列底层通常使用堆（Heap）数据结构来实现。堆是一种特殊的完全二叉树，分为最大堆和最小堆。最大堆的每个节点的值都大于或等于其子节点的值，最小堆则相反。
  
- **容器适配器**：`priority_queue` 是一个容器适配器，它基于其他容器（如 `vector` 或 `deque`）来实现其功能。默认情况下，`priority_queue` 使用 `vector` 作为底层容器。

### 2. 优先队列的基本操作

- **插入元素**：使用 `push()` 方法将元素插入优先队列。插入操作的时间复杂度为 O(log n)，因为需要维护堆的性质。
  
- **访问顶部元素**：使用 `top()` 方法访问优先队列中的顶部元素（即优先级最高的元素）。访问操作的时间复杂度为 O(1)。
  
- **删除顶部元素**：使用 `pop()` 方法删除优先队列中的顶部元素。删除操作的时间复杂度为 O(log n)，因为需要重新调整堆。
  
- **检查队列是否为空**：使用 `empty()` 方法检查优先队列是否为空。
  
- **获取队列大小**：使用 `size()` 方法获取优先队列中元素的数量。

### 3. 优先队列的声明

`priority_queue` 的声明如下：

```cpp
template <class T, class Container = vector<T>, class Compare = less<typename Container::value_type>>
class priority_queue;
```

- **T**：存储的元素类型。
- **Container**：底层容器类型，默认为 `vector<T>`。
- **Compare**：比较函数对象类型，默认为 `less<T>`，即最大堆。

### 4. 默认行为（最大堆）

默认情况下，`priority_queue` 是一个最大堆，即顶部元素是队列中的最大元素。

```cpp
#include <iostream>
#include <queue>

int main() {
    std::priority_queue<int> pq;

    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";  // 输出: 50 30 20 10
        pq.pop();
    }

    return 0;
}
```

### 5. 自定义比较函数（最小堆）

如果你想实现一个最小堆，可以通过自定义比较函数来实现。你可以使用 `greater<T>` 作为比较函数。

```cpp
#include <iostream>
#include <queue>
#include <functional>  // for greater

int main() {
    std::priority_queue<int, std::vector<int>, std::greater<int>> pq;

    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";  // 输出: 10 20 30 50
        pq.pop();
    }

    return 0;
}
```

### 6. 自定义比较函数

你还可以通过自定义比较函数来实现更复杂的优先级规则。比较函数应该是一个可调用对象（如函数指针、函数对象或 lambda 表达式），它接受两个参数并返回一个布尔值，表示第一个参数是否应该排在第二个参数之前。

```cpp
#include <iostream>
#include <queue>

struct CustomCompare {
    bool operator()(int a, int b) {
        return a > b;  // 最小堆
        // 如果返回true，那么将a放到b的下面
    }
};

int main() {
    std::priority_queue<int, std::vector<int>, CustomCompare> pq;

    pq.push(30);
    pq.push(10);
    pq.push(50);
    pq.push(20);

    while (!pq.empty()) {
        std::cout << pq.top() << " ";  // 输出: 10 20 30 50
        pq.pop();
    }

    return 0;
}
```

### 7. 使用自定义对象

优先队列不仅可以存储基本数据类型，还可以存储自定义对象。你需要为自定义对象定义比较函数或重载比较运算符。

```cpp
#include <iostream>
#include <queue>
#include <string>

struct Person {
    std::string name;
    int age;

    Person(std::string n, int a) : name(n), age(a) {}

    // 重载小于运算符，用于最大堆
    bool operator<(const Person& other) const {
        return age < other.age;
    }
};

int main() {
    std::priority_queue<Person> pq;

    pq.push(Person("Alice", 30));
    pq.push(Person("Bob", 20));
    pq.push(Person("Charlie", 40));

    while (!pq.empty()) {
        std::cout << pq.top().name << " " << pq.top().age << std::endl;
        pq.pop();
    }

    return 0;
}
```

### 8. 总结

- `priority_queue` 是一个基于堆的容器适配器，默认情况下是最大堆。
- 你可以通过自定义比较函数来实现最小堆或其他优先级规则。
- `priority_queue` 提供了 `push()`、`pop()`、`top()`、`empty()` 和 `size()` 等基本操作。
- 优先队列适用于需要频繁访问和删除最高优先级元素的场景，如任务调度、Dijkstra 算法等。

希望这些内容能帮助你更好地理解和使用 C++ 中的优先队列！