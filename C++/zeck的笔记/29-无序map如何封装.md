---
title: unorderedmap以及手写无序map
date: 2025-01-14 16:18:11
tags: C++ cppbase
categories: C++ cppbase
---

# unordermap用法

`unordered_map` 是 C++ 标准库中的关联容器，提供了基于哈希表的键值对存储结构。与 `map` （基于红黑树实现）不同，`unordered_map` 提供的是平均常数时间复杂度的查找、插入和删除操作，但不保证元素的顺序。



以下是 `unordered_map` 的详细用法说明：

## 1. 头文件



要使用 `unordered_map`，需要包含头文件：



```cpp
#include <unordered_map>
```



## 2. 基本定义



`unordered_map` 的基本模板定义如下：



```cpp
std::unordered_map<KeyType, ValueType, Hash = std::hash<KeyType>, KeyEqual = std::equal_to<KeyType>, Allocator = std::allocator<std::pair<const KeyType, ValueType>>>
```



### 常用模板参数：



- **KeyType**：键的类型，需要支持哈希运算和相等比较。
- **ValueType**：值的类型。
- **Hash**：哈希函数，默认为 `std::hash<KeyType>`。
- **KeyEqual**：键相等的比较函数，默认为 `std::equal_to<KeyType>`。
- **Allocator**：内存分配器，默认为 `std::allocator`。



## 3. 常用操作



### 3.1 创建和初始化



```cpp
#include <iostream>
#include <unordered_map>
#include <string>

int main() {
    // 创建一个空的 unordered_map，键为 string，值为 int
    std::unordered_map<std::string, int> umap;

    // 使用初始化列表初始化
    std::unordered_map<std::string, int> umap_init = {
        {"apple", 3},
        {"banana", 5},
        {"orange", 2}
    };

    // 使用其他容器范围初始化
    std::map<std::string, int> ordered_map = {{"carrot", 4}, {"lettuce", 1}};
    std::unordered_map<std::string, int> umap_from_map(ordered_map.begin(), ordered_map.end());

    return 0;
}
```



### 3.2 插入元素



```cpp
// 方法1：使用下标操作符
umap["grape"] = 7;

// 方法2：使用 insert
umap.insert({"melon", 6});

// 方法3：使用 emplace，直接在容器内部构造元素
umap.emplace("kiwi", 4);
```



### 3.3 访问元素



```cpp
// 使用下标操作符访问或插入
int apple_count = umap["apple"]; // 如果 "apple" 不存在，会插入一个默认值

// 使用 at() 方法访问，不存在时会抛出异常
try {
    int banana_count = umap.at("banana");
} catch (const std::out_of_range& e) {
    std::cerr << "Key not found." << std::endl;
}

// 使用 find() 方法查找
auto it = umap.find("orange");
if (it != umap.end()) {
    std::cout << "Orange count: " << it->second << std::endl;
} else {
    std::cout << "Orange not found." << std::endl;
}
```



### 3.4 删除元素



```cpp
// 根据键删除
umap.erase("grape");

// 根据迭代器删除
auto it = umap.find("banana");
if (it != umap.end()) {
    umap.erase(it);
}

// 清空整个容器
umap.clear();
```



### 3.5 遍历元素



```cpp
for (const auto& pair : umap) {
    std::cout << pair.first << ": " << pair.second << std::endl;
}

// 使用迭代器
for (auto it = umap.begin(); it != umap.end(); ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```



### 3.6 其他常用方法



```cpp
// 获取大小
size_t size = umap.size();

// 检查是否为空
bool is_empty = umap.empty();

// 获取桶的数量（用于哈希表内部结构）
size_t bucket_count = umap.bucket_count();

// 重新哈希，调整桶的数量
umap.rehash(20);

// 从一个容器中移交元素到另一个容器
std::unordered_map<std::string, int> umap2 = std::move(umap);
```



## 4. 性能优化



### 4.1 预分配桶数



如果预先知道元素的大致数量，可以通过 `reserve` 预分配内存，以减少哈希表的重哈希开销：



```cpp
umap.reserve(100); // 预分配足够容纳100个元素的桶
```



### 4.2 自定义哈希函数



如果键的类型是自定义类型或需要特殊的哈希策略，可以自定义哈希函数。例如，自定义结构体作为键：



```cpp
struct Point {
    int x;
    int y;

    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

// 自定义哈希函数
struct PointHash {
    std::size_t operator()(const Point& p) const {
        return std::hash<int>()(p.x) ^ (std::hash<int>()(p.y) << 1);
    }
};

// 定义 unordered_map 使用自定义哈希函数
std::unordered_map<Point, std::string, PointHash> point_map;
point_map[{1, 2}] = "A";
point_map[{3, 4}] = "B";
```



### 4.3 自定义键相等比较



如果需要自定义键的比较逻辑，可以提供自定义的 `KeyEqual` 函数对象：



```cpp
struct PointEqual {
    bool operator()(const Point& a, const Point& b) const {
        return (a.x == b.x) && (a.y == b.y);
    }
};

// 定义 unordered_map 使用自定义哈希和比较函数
std::unordered_map<Point, std::string, PointHash, PointEqual> point_map;
```



## 5. 与 `map` 的比较



- **底层实现**：`unordered_map` 基于哈希表，实现的操作平均时间复杂度为常数级别；`map` 基于红黑树，实现的查找、插入、删除操作时间复杂度为对数级别。
- **元素顺序**：`unordered_map` 不保证元素的顺序；`map` 按键的顺序（通常是升序）存储元素。
- **适用场景**：当需要快速查找、插入和删除，且不关心元素顺序时，选择 `unordered_map`；当需要有序存储或按顺序遍历时，选择 `map`。



## 6. 完整示例



以下是一个使用 `unordered_map` 的完整示例：



```cpp
#include <iostream>
#include <unordered_map>
#include <string>

// 自定义类型
struct Person {
    std::string name;
    int age;

    bool operator==(const Person& other) const {
        return name == other.name && age == other.age;
    }
};

// 自定义哈希函数
struct PersonHash {
    std::size_t operator()(const Person& p) const {
        return std::hash<std::string>()(p.name) ^ (std::hash<int>()(p.age) << 1);
    }
};

int main() {
    // 创建一个 unordered_map，键为 string，值为 int
    std::unordered_map<std::string, int> fruit_count;
    fruit_count["apple"] = 5;
    fruit_count["banana"] = 3;
    fruit_count.emplace("orange", 2);

    // 访问元素
    std::cout << "Apple count: " << fruit_count["apple"] << std::endl;

    // 遍历
    for (const auto& pair : fruit_count) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    // 使用自定义类型作为键
    std::unordered_map<Person, std::string, PersonHash> person_map;
    Person p1{"Alice", 30};
    Person p2{"Bob", 25};
    person_map[p1] = "Engineer";
    person_map.emplace(p2, "Designer");

    // 访问自定义类型键的值
    Person p3{"Alice", 30};
    std::cout << "Alice's job: " << person_map[p3] << std::endl;

    return 0;
}
```



**输出示例**：



```yaml
Apple count: 5
banana: 3
orange: 2
apple: 5
Alice's job: Engineer
```

# 手写unordermap

## 1. 哈希表的基本原理



哈希表是一种基于键值对的数据结构，通过**哈希函数**（Hash Function）将键映射到表中的一个索引位置，以实现快速的数据访问。哈希表的关键特性包括：



- **哈希函数**：将键映射到表中一个特定的桶（Bucket）或槽（Slot）。
- **冲突解决**：当不同的键通过哈希函数映射到同一个桶时，需要一种机制来处理这些冲突。常见的方法有**链地址法**（Separate Chaining）和**开放地址法**（Open Addressing）。
- **负载因子**（Load Factor）：表示表中已存储元素的数量与表大小之间的比率。高负载因子可能导致更多的冲突，需要通过扩容来维持性能。



在本实现中，我们将采用**链地址法**来处理哈希冲突，即每个桶存储一个链表（或其他动态数据结构）来存储具有相同哈希值的元素。



## 2. 数据结构设计



### HashNode 结构



`HashNode` 用于存储键值对及相关的指针，以构建链表。每个 `HashNode` 包含键、值和指向下一个节点的指针。

``` cpp
#include <vector>
#include <list>
#include <utility> // For std::pair
#include <functional> // For std::hash
#include <iterator> // For iterator_traits
#include <stdexcept> // For exceptions

// HashNode 结构定义
template <typename Key, typename T>
struct HashNode {
    std::pair<const Key, T> data;
    HashNode* next;

    HashNode(const Key& key, const T& value)
        : data(std::make_pair(key, value)), next(nullptr) {}
};
```

### MyHashMap 类定义



`MyHashMap` 是我们自定义的哈希表实现，支持基本的 `Map` 操作和迭代器功能。它使用一个向量（`std::vector`）来存储桶，每个桶是一个链表，用于处理冲突。



```cpp
template <typename Key, typename T, typename Hash = std::hash<Key>>
class MyHashMap {
public:
    // 迭代器类前向声明
    class Iterator;

    // 类型定义
    using key_type = Key;
    using mapped_type = T;
    using value_type = std::pair<const Key, T>;
    using size_type = size_t;

    // 构造函数及析构函数
    MyHashMap(size_type initial_capacity = 16, double max_load_factor = 0.75);
    ~MyHashMap();

    // 禁止拷贝构造和赋值
    MyHashMap(const MyHashMap&) = delete;
    MyHashMap& operator=(const MyHashMap&) = delete;

    // 基本操作
    void insert(const Key& key, const T& value);
    T* find(const Key& key);
    const T* find(const Key& key) const;
    bool erase(const Key& key);
    size_type size() const;
    bool empty() const;
    void clear();

    // 迭代器操作
    Iterator begin();
    Iterator end();

    // 迭代器类
    class Iterator {
    public:
        // 迭代器别名
        using iterator_category = std::forward_iterator_tag;
        using value_type = std::pair<const Key, T>;
        using difference_type = std::ptrdiff_t;
        using pointer = value_type*;
        using reference = value_type&;

        // 构造函数
        Iterator(MyHashMap* map, size_type bucket_index, HashNode<Key, T>* node);

        // 解引用操作符
        reference operator*() const;
        pointer operator->() const;

        // 递增操作符
        Iterator& operator++();
        Iterator operator++(int);

        // 比较操作符
        bool operator==(const Iterator& other) const;
        bool operator!=(const Iterator& other) const;

    private:
        MyHashMap* map_;
        size_type bucket_index_;
        HashNode<Key, T>* current_node_;

        // 移动到下一个有效节点
        void advance();
    };

private:
    std::vector<HashNode<Key, T>*> buckets_;
    size_type bucket_count_;
    size_type element_count_;
    double max_load_factor_;
    Hash hash_func_;

    // 辅助函数
    void rehash();
};
```



## 3. 基本操作实现



### 构造函数及析构函数



初始化哈希表，设置初始容量和负载因子。



```cpp
// 构造函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::MyHashMap(size_type initial_capacity, double max_load_factor)
    : bucket_count_(initial_capacity),
      element_count_(0),
      max_load_factor_(max_load_factor),
      hash_func_(Hash()) {
    buckets_.resize(bucket_count_, nullptr);
}

// 析构函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::~MyHashMap() {
    clear();
}
```



### 插入（Insert）



向哈希表中插入键值对。如果键已存在，则更新其值。



```cpp
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::insert(const Key& key, const T& value) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            node->data.second = value; // 更新值
            return;
        }
        node = node->next;
    }

    // 键不存在，插入新节点到链表头部
    HashNode<Key, T>* new_node = new HashNode<Key, T>(key, value);
    new_node->next = buckets_[index];
    buckets_[index] = new_node;
    ++element_count_;

    // 检查负载因子，可能需要扩容
    double load_factor = static_cast<double>(element_count_) / bucket_count_;
    if (load_factor > max_load_factor_) {
        rehash();
    }
}
```



### 查找（Find）



根据键查找对应的值，返回指向值的指针。如果未找到，则返回 `nullptr`。



```cpp
template <typename Key, typename T, typename Hash>
T* MyHashMap<Key, T, Hash>::find(const Key& key) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            return &(node->data.second);
        }
        node = node->next;
    }
    return nullptr;
}

template <typename Key, typename T, typename Hash>
const T* MyHashMap<Key, T, Hash>::find(const Key& key) const {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            return &(node->data.second);
        }
        node = node->next;
    }
    return nullptr;
}
```



### 删除（Erase）



根据键删除对应的键值对，返回删除是否成功。



```cpp
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::erase(const Key& key) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    HashNode<Key, T>* prev = nullptr;

    while (node != nullptr) {
        if (node->data.first == key) {
            if (prev == nullptr) {
                buckets_[index] = node->next;
            } else {
                prev->next = node->next;
            }
            delete node;
            --element_count_;
            return true;
        }
        prev = node;
        node = node->next;
    }
    return false; // 未找到键
}
```



### 清空（Clear）



删除哈希表中的所有元素，释放内存。



```cpp
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::clear() {
    for (size_type i = 0; i < bucket_count_; ++i) {
        HashNode<Key, T>* node = buckets_[i];
        while (node != nullptr) {
            HashNode<Key, T>* temp = node;
            node = node->next;
            delete temp;
        }
        buckets_[i] = nullptr;
    }
    element_count_ = 0;
}
```



### 动态扩容（Rehashing）



当负载因子超过阈值时，扩展哈希表容量并重新分配所有元素。



```cpp
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::rehash() {
    size_type new_bucket_count = bucket_count_ * 2;
    std::vector<HashNode<Key, T>*> new_buckets(new_bucket_count, nullptr);

    // 重新分配所有元素
    for (size_type i = 0; i < bucket_count_; ++i) {
        HashNode<Key, T>* node = buckets_[i];
        while (node != nullptr) {
            HashNode<Key, T>* next_node = node->next;
            size_type new_index = hash_func_(node->data.first) % new_bucket_count;

            // 插入到新桶的头部
            node->next = new_buckets[new_index];
            new_buckets[new_index] = node;

            node = next_node;
        }
    }

    // 替换旧桶
    buckets_ = std::move(new_buckets);
    bucket_count_ = new_bucket_count;
}
```



### 获取大小和状态



```cpp
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::size_type MyHashMap<Key, T, Hash>::size() const {
    return element_count_;
}

template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::empty() const {
    return element_count_ == 0;
}
```



## 4. 迭代器的实现



为了支持迭代器操作，使 `MyHashMap` 能够像标准容器一样被遍历，我们需要实现一个内部的 `Iterator` 类。



### Iterator 类定义



`Iterator` 类需要跟踪当前桶的索引和当前节点指针。它还需要能够找到下一个有效的节点。



```cpp
// Iterator 构造函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::Iterator::Iterator(MyHashMap* map, size_type bucket_index, HashNode<Key, T>* node)
    : map_(map), bucket_index_(bucket_index), current_node_(node) {}

// 解引用操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator::reference
MyHashMap<Key, T, Hash>::Iterator::operator*() const {
    if (current_node_ == nullptr) {
        throw std::out_of_range("Iterator out of range");
    }
    return current_node_->data;
}

// 成员访问操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator::pointer
MyHashMap<Key, T, Hash>::Iterator::operator->() const {
    if (current_node_ == nullptr) {
        throw std::out_of_range("Iterator out of range");
    }
    return &(current_node_->data);
}

// 前置递增操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator&
MyHashMap<Key, T, Hash>::Iterator::operator++() {
    advance();
    return *this;
}

// 后置递增操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::Iterator::operator++(int) {
    Iterator temp = *this;
    advance();
    return temp;
}

// 比较操作符==
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::Iterator::operator==(const Iterator& other) const {
    return map_ == other.map_ &&
           bucket_index_ == other.bucket_index_ &&
           current_node_ == other.current_node_;
}

// 比较操作符!=
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::Iterator::operator!=(const Iterator& other) const {
    return !(*this == other);
}

// advance 函数：移动到下一个有效节点
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::Iterator::advance() {
    if (current_node_ != nullptr) {
        current_node_ = current_node_->next;
    }

    while (current_node_ == nullptr && bucket_index_ + 1 < map_->bucket_count_) {
        ++bucket_index_;
        current_node_ = map_->buckets_[bucket_index_];
    }
}
```



### 迭代器操作



在 `MyHashMap` 类中实现 `begin()` 和 `end()` 函数来返回迭代器。



```cpp
// begin() 函数
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::begin() {
    for (size_type i = 0; i < bucket_count_; ++i) {
        if (buckets_[i] != nullptr) {
            return Iterator(this, i, buckets_[i]);
        }
    }
    return end();
}

// end() 函数
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::end() {
    return Iterator(this, bucket_count_, nullptr);
}
```



## 5. 完整代码示例



以下是完整的 `MyHashMap` 实现，包括所有上述内容：



```cpp
#include <vector>
#include <list>
#include <utility> // For std::pair
#include <functional> // For std::hash
#include <iterator> // For iterator_traits
#include <stdexcept> // For exceptions
#include <iostream>

// HashNode 结构定义
template <typename Key, typename T>
struct HashNode {
    std::pair<const Key, T> data;
    HashNode* next;

    HashNode(const Key& key, const T& value)
        : data(std::make_pair(key, value)), next(nullptr) {}
};

// MyHashMap 类定义
template <typename Key, typename T, typename Hash = std::hash<Key>>
class MyHashMap {
public:
    // 迭代器类前向声明
    class Iterator;

    // 类型定义
    using key_type = Key;
    using mapped_type = T;
    using value_type = std::pair<const Key, T>;
    using size_type = size_t;

    // 构造函数及析构函数
    MyHashMap(size_type initial_capacity = 16, double max_load_factor = 0.75);
    ~MyHashMap();

    // 禁止拷贝构造和赋值
    MyHashMap(const MyHashMap&) = delete;
    MyHashMap& operator=(const MyHashMap&) = delete;

    // 基本操作
    void insert(const Key& key, const T& value);
    T* find(const Key& key);
    const T* find(const Key& key) const;
    bool erase(const Key& key);
    size_type size() const;
    bool empty() const;
    void clear();

    // 迭代器操作
    Iterator begin();
    Iterator end();

    // 迭代器类
    class Iterator {
    public:
        // 迭代器别名
        using iterator_category = std::forward_iterator_tag;
        using value_type = std::pair<Key, T>;
        using difference_type = std::ptrdiff_t;
        using pointer = value_type*;
        using reference = value_type&;

        // 构造函数
        Iterator(MyHashMap* map, size_type bucket_index, HashNode<Key, T>* node);

        // 解引用操作符
        reference operator*() const;
        pointer operator->() const;

        // 递增操作符
        Iterator& operator++();
        Iterator operator++(int);

        // 比较操作符
        bool operator==(const Iterator& other) const;
        bool operator!=(const Iterator& other) const;

    private:
        MyHashMap* map_;
        size_type bucket_index_;
        HashNode<Key, T>* current_node_;

        // 移动到下一个有效节点
        void advance();
    };

private:
    std::vector<HashNode<Key, T>*> buckets_;
    size_type bucket_count_;
    size_type element_count_;
    double max_load_factor_;
    Hash hash_func_;

    // 辅助函数
    void rehash();
};

// 构造函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::MyHashMap(size_type initial_capacity, double max_load_factor)
    : bucket_count_(initial_capacity),
      element_count_(0),
      max_load_factor_(max_load_factor),
      hash_func_(Hash()) {
    buckets_.resize(bucket_count_, nullptr);
}

// 析构函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::~MyHashMap() {
    clear();
}

// 插入函数
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::insert(const Key& key, const T& value) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            node->data.second = value; // 更新值
            return;
        }
        node = node->next;
    }

    // 键不存在，插入新节点到链表头部
    HashNode<Key, T>* new_node = new HashNode<Key, T>(key, value);
    new_node->next = buckets_[index];
    buckets_[index] = new_node;
    ++element_count_;

    // 检查负载因子，可能需要扩容
    double load_factor = static_cast<double>(element_count_) / bucket_count_;
    if (load_factor > max_load_factor_) {
        rehash();
    }
}

// 查找函数（非常量版本）
template <typename Key, typename T, typename Hash>
T* MyHashMap<Key, T, Hash>::find(const Key& key) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            return &(node->data.second);
        }
        node = node->next;
    }
    return nullptr;
}

// 查找函数（常量版本）
template <typename Key, typename T, typename Hash>
const T* MyHashMap<Key, T, Hash>::find(const Key& key) const {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    while (node != nullptr) {
        if (node->data.first == key) {
            return &(node->data.second);
        }
        node = node->next;
    }
    return nullptr;
}

// 删除函数
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::erase(const Key& key) {
    size_type hash_value = hash_func_(key);
    size_type index = hash_value % bucket_count_;

    HashNode<Key, T>* node = buckets_[index];
    HashNode<Key, T>* prev = nullptr;

    while (node != nullptr) {
        if (node->data.first == key) {
            if (prev == nullptr) {
                buckets_[index] = node->next;
            } else {
                prev->next = node->next;
            }
            delete node;
            --element_count_;
            return true;
        }
        prev = node;
        node = node->next;
    }
    return false; // 未找到键
}

// 清空函数
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::clear() {
    for (size_type i = 0; i < bucket_count_; ++i) {
        HashNode<Key, T>* node = buckets_[i];
        while (node != nullptr) {
            HashNode<Key, T>* temp = node;
            node = node->next;
            delete temp;
        }
        buckets_[i] = nullptr;
    }
    element_count_ = 0;
}

// 动态扩容函数
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::rehash() {
    size_type new_bucket_count = bucket_count_ * 2;
    std::vector<HashNode<Key, T>*> new_buckets(new_bucket_count, nullptr);

    // 重新分配所有元素
    for (size_type i = 0; i < bucket_count_; ++i) {
        HashNode<Key, T>* node = buckets_[i];
        while (node != nullptr) {
            HashNode<Key, T>* next_node = node->next;
            size_type new_index = hash_func_(node->data.first) % new_bucket_count;

            // 插入到新桶的头部
            node->next = new_buckets[new_index];
            new_buckets[new_index] = node;

            node = next_node;
        }
    }

    // 替换旧桶
    buckets_ = std::move(new_buckets);
    bucket_count_ = new_bucket_count;
}

// begin() 函数
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::begin() {
    for (size_type i = 0; i < bucket_count_; ++i) {
        if (buckets_[i] != nullptr) {
            return Iterator(this, i, buckets_[i]);
        }
    }
    return end();
}

// end() 函数
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::end() {
    return Iterator(this, bucket_count_, nullptr);
}

// Iterator 构造函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::Iterator::Iterator(MyHashMap* map, size_type bucket_index, HashNode<Key, T>* node)
    : map_(map), bucket_index_(bucket_index), current_node_(node) {}

// 解引用操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator::reference
MyHashMap<Key, T, Hash>::Iterator::operator*() const {
    if (current_node_ == nullptr) {
        throw std::out_of_range("Iterator out of range");
    }
    return current_node_->data;
}

// 成员访问操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator::pointer
MyHashMap<Key, T, Hash>::Iterator::operator->() const {
    if (current_node_ == nullptr) {
        throw std::out_of_range("Iterator out of range");
    }
    return &(current_node_->data);
}

// 前置递增操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator&
MyHashMap<Key, T, Hash>::Iterator::operator++() {
    advance();
    return *this;
}

// 后置递增操作符
template <typename Key, typename T, typename Hash>
typename MyHashMap<Key, T, Hash>::Iterator
MyHashMap<Key, T, Hash>::Iterator::operator++(int) {
    Iterator temp = *this;
    advance();
    return temp;
}

// 比较操作符==
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::Iterator::operator==(const Iterator& other) const {
    return map_ == other.map_ &&
           bucket_index_ == other.bucket_index_ &&
           current_node_ == other.current_node_;
}

// 比较操作符!=
template <typename Key, typename T, typename Hash>
bool MyHashMap<Key, T, Hash>::Iterator::operator!=(const Iterator& other) const {
    return !(*this == other);
}

// advance 函数：移动到下一个有效节点
template <typename Key, typename T, typename Hash>
void MyHashMap<Key, T, Hash>::Iterator::advance() {
   if(current_node_ != nullptr){
         current_node_ = current_node_->next;
    }
   while(current_node_ == nullptr){
        if(bucket_index_ +1  < map_->bucket_count_){
                   ++bucket_index_;
                   current_node_ = map_->buckets_[bucket_index_];
          }else if(bucket_index_ +1  == map_->bucket_count_){
                   ++bucket_index_;
                   break;
           }
       }	
}
```



## 6. 使用示例



以下是一个使用 `MyHashMap` 的示例，展示如何插入、查找、删除以及使用迭代器遍历元素。



```cpp
int main() {
    MyHashMap<std::string, int> myMap;

    // 插入元素
    myMap.insert("apple", 3);
    myMap.insert("banana", 5);
    myMap.insert("orange", 2);
    myMap.insert("grape", 7);
    myMap.insert("cherry", 4);

    // 使用迭代器遍历元素
    std::cout << "Map contents:\n";
    for(auto it = myMap.begin(); it != myMap.end(); ++it) {
        std::cout << it->first << " => " << it->second << "\n";
    }

    // 查找元素
    std::string keyToFind = "banana";
    int* value = myMap.find(keyToFind);
    if(value != nullptr) {
        std::cout << "\nFound " << keyToFind << " with value: " << *value << "\n";
    } else {
        std::cout << "\n" << keyToFind << " not found.\n";
    }

    // 删除元素
    myMap.erase("apple");
    myMap.erase("cherry");

    // 再次遍历
    std::cout << "\nAfter erasing apple and cherry:\n";
    for(auto it = myMap.begin(); it != myMap.end(); ++it) {
        std::cout << it->first << " => " << it->second << "\n";
    }

    return 0;
}
```



### 预期输出



```makefile
Map contents:
cherry => 4
banana => 5
apple => 3
grape => 7
orange => 2

Found banana with value: 5

After erasing apple and cherry:
banana => 5
grape => 7
orange => 2
```



**注意**：由于哈希表的桶顺序依赖于哈希函数的实现，输出顺序可能与预期有所不同。

`Hash()` 在上述 `MyHashMap` 实现中是一个**哈希函数对象**。让我们详细解释一下它的含义以及它在代码中的作用。



## `Hash` 是什么？



在 `MyHashMap` 的模板定义中，`Hash` 是一个**模板参数**，用于指定键类型 `Key` 的哈希函数。它有一个默认值 `std::hash<Key>`，这意味着如果用户在实例化 `MyHashMap` 时没有提供自定义的哈希函数，`std::hash<Key>` 将被使用。



```cpp
template <typename Key, typename T, typename Hash = std::hash<Key>>
class MyHashMap {
    // ...
};
```



### 默认情况下：`std::hash<Key>`



`std::hash` 是 C++ 标准库（STL）中提供的一个模板结构，用于为各种内置类型（如 `int`, `std::string` 等）生成哈希值。`std::hash<Key>` 会根据 `Key` 的类型自动选择合适的哈希函数实现。



例如：



- 对于 `int` 类型，`std::hash<int>` 会生成一个简单的哈希值。
- 对于 `std::string` 类型，`std::hash<std::string>` 会基于字符串内容生成哈希值。



```cpp
std::hash<int> intHasher;
size_t hashValue = intHasher(42); // 生成整数 42 的哈希值

std::hash<std::string> stringHasher;
size_t stringHash = stringHasher("hello"); // 生成字符串 "hello" 的哈希值
```



### 自定义哈希函数



除了使用 `std::hash`，用户还可以自定义哈希函数，以适应特定的需求或优化性能。例如，假设你有一个自定义的键类型 `Point`：



```cpp
struct Point {
    int x;
    int y;

    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};
```



你可以定义一个自定义的哈希函数 `PointHasher`：



```cpp
struct PointHasher {
    size_t operator()(const Point& p) const {
        // 简单的哈希组合，实际应用中应选择更好的哈希组合方法
        return std::hash<int>()(p.x) ^ (std::hash<int>()(p.y) << 1);
    }
};
```



然后，在实例化 `MyHashMap` 时使用自定义哈希函数：



```cpp
MyHashMap<Point, std::string, PointHasher> pointMap;
Point p1{1, 2};
pointMap.insert(p1, "Point1");
```



## `Hash()` 在代码中的作用



在 `MyHashMap` 的构造函数中，`Hash()` 用于**实例化**哈希函数对象，并将其赋值给成员变量 `hash_func_`：



```cpp
// 构造函数
template <typename Key, typename T, typename Hash>
MyHashMap<Key, T, Hash>::MyHashMap(size_type initial_capacity, double max_load_factor)
    : bucket_count_(initial_capacity),
      element_count_(0),
      max_load_factor_(max_load_factor),
      hash_func_(Hash()) { // 这里的 Hash() 是一个默认构造函数调用
    buckets_.resize(bucket_count_, nullptr);
}
```



### 具体作用：



1. **实例化哈希函数对象**：
   - `Hash()` 会调用 `Hash` 类型的默认构造函数，创建一个哈希函数对象。
   - 如果 `Hash` 是 `std::hash<Key>`，那么就创建一个 `std::hash<Key>` 对象。
2. **存储哈希函数对象**：
   - 生成的哈希函数对象被存储在成员变量 `hash_func_` 中，以便在哈希表的各种操作（如插入、查找、删除）中使用。
3. **支持不同的哈希函数**：
   - 由于 `Hash` 是一个模板参数，可以灵活地使用不同的哈希函数，无需修改 `MyHashMap` 的内部实现。



## 示例代码解释



以下是相关部分的简化示例，帮助理解 `Hash` 和 `Hash()` 的作用：



```cpp
#include <vector>
#include <functional> // For std::hash

template <typename Key, typename T, typename Hash = std::hash<Key>>
class MyHashMap {
public:
    MyHashMap(size_t initial_capacity = 16, double max_load_factor = 0.75)
        : bucket_count_(initial_capacity),
          element_count_(0),
          max_load_factor_(max_load_factor),
          hash_func_(Hash()) { // 实例化哈希函数对象
        buckets_.resize(bucket_count_, nullptr);
    }

    void insert(const Key& key, const T& value) {
        size_t hash_value = hash_func_(key); // 使用哈希函数对象
        size_t index = hash_value % bucket_count_;
        // 插入逻辑...
    }

private:
    std::vector<HashNode<Key, T>*> buckets_;
    size_t bucket_count_;
    size_t element_count_;
    double max_load_factor_;
    Hash hash_func_; // 存储哈希函数对象
};
```



## 总结



- **`Hash` 是模板参数**，用于指定键类型 `Key` 的哈希函数。默认情况下，它使用 C++ 标准库中的 `std::hash<Key>`。
- **`Hash()` 是一个默认构造函数调用**，用于实例化哈希函数对象，并将其存储在 `hash_func_` 成员变量中，以便在哈希表操作中使用。
- **用户可以自定义哈希函数**，通过提供自定义的哈希函数对象，实现对特定键类型的优化或满足特殊需求。

