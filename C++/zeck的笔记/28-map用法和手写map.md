

# std::map用法

`std::map` 是 C++ 标准模板库（STL）中的一个关联容器，用于存储键值对（key-value pairs），其中每个键都是唯一的，并且按照特定的顺序（通常是升序）自动排序。`std::map` 通常基于红黑树实现，提供对元素的高效查找、插入和删除操作。



## 1. 基本特性



- **有序性**：`std::map` 中的元素按照键的顺序自动排序，默认使用 `<` 运算符进行比较。
- **唯一键**：每个键在 `std::map` 中必须是唯一的，如果尝试插入重复的键，则插入操作会失败。
- **关联容器**：通过键快速访问对应的值，通常具有对数时间复杂度（O(log n)）。
- **可变性**：可以动态地插入和删除元素。



## 2. 头文件和命名空间



要使用 `std::map`，需要包含头文件 `<map>` 并使用 `std` 命名空间：



```cpp
#include <map>
#include <iostream>
#include <string>

using namespace std;
```



## 3. 声明和初始化



### 3.1 声明一个 `std::map`



```cpp
// 键为 int，值为 std::string 的 map
map<int, string> myMap;

// 键为 std::string，值为 double 的 map
map<string, double> priceMap;
```



### 3.2 初始化 `std::map`



可以使用初始化列表或其他方法初始化 `map`：



```cpp
map<int, string> myMap = {
    {1, "Apple"},
    {2, "Banana"},
    {3, "Cherry"}
};
```



## 4. 主要操作



### 4.1 插入元素



有几种方法可以向 `std::map` 中插入元素：



#### 4.1.1 使用 `insert` 函数



```cpp
myMap.insert(pair<int, string>(4, "Date"));
// 或者使用 `make_pair`
myMap.insert(make_pair(5, "Elderberry"));
// 或者使用初始化列表
myMap.insert({6, "Fig"});
```



#### 4.1.2 使用下标运算符 `[]`



```cpp
myMap[7] = "Grape";
// 如果键 8 不存在，则会插入键 8 并赋值
myMap[8] = "Honeydew";
```



**注意**：使用 `[]` 运算符时，如果键不存在，会自动插入该键，并将值初始化为类型的默认值。



### 4.2 访问元素



#### 4.2.1 使用下标运算符 `[]`



```cpp
string fruit = myMap[1]; // 获取键为 1 的值 "Apple"
```



**注意**：如果键不存在，`[]` 会插入该键并返回默认值。



#### 4.2.2 使用 `at` 成员函数



```cpp
try {
    string fruit = myMap.at(2); // 获取键为 2 的值 "Banana"
} catch (const out_of_range& e) {
    cout << "Key not found." << endl;
}
```



`at` 函数在键不存在时会抛出 `std::out_of_range` 异常，适合需要异常处理的场景。



#### 4.2.3 使用 `find` 成员函数



```cpp
auto it = myMap.find(3);
if (it != myMap.end()) {
    cout << "Key 3: " << it->second << endl; // 输出 "Cherry"
} else {
    cout << "Key 3 not found." << endl;
}
```



`find` 返回一个迭代器，指向找到的元素，若未找到则返回 `map::end()`。



### 4.3 删除元素



#### 4.3.1 使用 `erase` 函数



```cpp
// 按键删除
myMap.erase(2);

// 按迭代器删除
auto it = myMap.find(3);
if (it != myMap.end()) {
    myMap.erase(it);
}

// 删除区间 [first, last)
myMap.erase(myMap.begin(), myMap.find(5));
```



#### 4.3.2 使用 `clear` 函数



```cpp
myMap.clear(); // 删除所有元素
```



### 4.4 遍历 `std::map`



#### 4.4.1 使用迭代器



```cpp
for (map<int, string>::iterator it = myMap.begin(); it != myMap.end(); ++it) {
    cout << "Key: " << it->first << ", Value: " << it->second << endl;
}
```



#### 4.4.2 使用基于范围的 `for` 循环（C++11 及以上）



```cpp
for (const auto& pair : myMap) {
    cout << "Key: " << pair.first << ", Value: " << pair.second << endl;
}
```



### 4.5 常用成员函数



- **`size()`**：返回容器中元素的数量。
- **`empty()`**：判断容器是否为空。
- **`count(key)`**：返回具有指定键的元素数量（对于 `map`，返回 0 或 1）。
- **`lower_bound(key)`** 和 **`upper_bound(key)`**：返回迭代器，分别指向第一个不小于和第一个大于指定键的元素。
- **`equal_range(key)`**：返回一个范围，包含所有等于指定键的元素。



## 5. 自定义键的排序



默认情况下，`std::map` 使用 `<` 运算符对键进行排序。如果需要自定义排序方式，可以提供一个自定义的比较函数或函数对象。



### 5.1 使用函数对象



```cpp
struct Compare {
    bool operator()(const int& a, const int& b) const {
        return a > b; // 降序排序
    }
};

map<int, string, Compare> myMapDesc;
myMapDesc[1] = "Apple";
myMapDesc[2] = "Banana";
// ...
```



### 5.2 使用 lambda 表达式（C++11 及以上）



需要注意，`std::map` 的第三个模板参数必须是可比较类型，不能直接使用 lambda 表达式作为模板参数。不过，可以使用 `std::function` 或自定义结构体来包装 lambda。



```cpp
struct CompareLambda {
    bool operator()(const int& a, const int& b) const {
        return a > b; // 降序排序
    }
};

map<int, string, CompareLambda> myMapDesc;
myMapDesc[1] = "Apple";
// ...
```



## 6. `std::map` 与其他关联容器的比较



- **`std::unordered_map`**：基于哈希表实现，提供平均常数时间复杂度的查找、插入和删除操作，但不保证元素的顺序。适用于对顺序无要求且需要高效查找的场景。
- **`std::multimap`**：允许多个相同键的元素，其他特性与 `std::map` 类似。适用于需要存储重复键值对的场景。



## 7. 性能考虑



- 时间复杂度

  ：

  - 查找、插入、删除：O(log n)
  - 遍历：O(n)

- **空间复杂度**：`std::map` 通常需要额外的空间来维护树结构，相比 `std::vector` 等序列容器，内存开销更大。



选择使用 `std::map` 还是其他容器，应根据具体需求和性能要求进行权衡。



## 8. 完整示例



以下是一个完整的示例，展示了 `std::map` 的基本用法：



```cpp
#include <iostream>
#include <map>
#include <string>

using namespace std;

int main() {
    // 创建一个 map，键为 int，值为 string
    map<int, string> myMap;

    // 插入元素
    myMap[1] = "Apple";
    myMap[2] = "Banana";
    myMap.insert({3, "Cherry"});
    myMap.insert(make_pair(4, "Date"));

    // 访问元素
    cout << "Key 1: " << myMap[1] << endl;
    cout << "Key 2: " << myMap.at(2) << endl;

    // 查找元素
    int keyToFind = 3;
    auto it = myMap.find(keyToFind);
    if (it != myMap.end()) {
        cout << "Found key " << keyToFind << ": " << it->second << endl;
    } else {
        cout << "Key " << keyToFind << " not found." << endl;
    }

    // 遍历 map
    cout << "All elements:" << endl;
    for (const auto& pair : myMap) {
        cout << "Key: " << pair.first << ", Value: " << pair.second << endl;
    }

    // 删除元素
    myMap.erase(2);
    cout << "After deleting key 2:" << endl;
    for (const auto& pair : myMap) {
        cout << "Key: " << pair.first << ", Value: " << pair.second << endl;
    }

    // 检查是否为空
    if (!myMap.empty()) {
        cout << "Map is not empty. Size: " << myMap.size() << endl;
    }

    // 清空所有元素
    myMap.clear();
    cout << "After clearing, map is " << (myMap.empty() ? "empty." : "not empty.") << endl;

    return 0;
}
```



**输出**：



```yaml
Key 1: Apple
Key 2: Banana
Found key 3: Cherry
All elements:
Key: 1, Value: Apple
Key: 2, Value: Banana
Key: 3, Value: Cherry
Key: 4, Value: Date
After deleting key 2:
Key: 1, Value: Apple
Key: 3, Value: Cherry
Key: 4, Value: Date
Map is not empty. Size: 3
After clearing, map is empty.
```

# BST实现map

## 1. 选择底层数据结构



`std::map` 通常基于平衡的二叉搜索树（如红黑树）实现，以保证操作的时间复杂度为 O(log n)。为了简化实现，本文将采用**普通的二叉搜索树**，即不进行自平衡处理。不过在实际应用中，为了保证性能，建议使用自平衡的树结构（例如红黑树、AVL 树）。



## 2. 设计数据结构



### 2.1 节点结构



首先，需要定义树的节点结构，每个节点包含键值对以及指向子节点的指针。



```cpp
#include <iostream>
#include <stack>
#include <utility> // For std::pair
#include <exception>

template <typename Key, typename T>
struct TreeNode {
    std::pair<Key, T> data;
    TreeNode* left;
    TreeNode* right;
    TreeNode* parent;

    TreeNode(const Key& key, const T& value, TreeNode* parentNode = nullptr)
        : data(std::make_pair(key, value)), left(nullptr), right(nullptr), parent(parentNode) {}
};
```



### 2.2 Map 类的定义



接下来，定义 `MyMap` 类，包含根节点和基本操作。



```cpp
template <typename Key, typename T>
class MyMap {
public:
    MyMap() : root(nullptr) {}
    ~MyMap() { clear(root); }

    // 禁止拷贝构造和赋值
    MyMap(const MyMap&) = delete;
    MyMap& operator=(const MyMap&) = delete;

    // 插入或更新键值对
    void insert(const Key& key, const T& value) {
        if (root == nullptr) {
            root = new TreeNode<Key, T>(key, value);
            return;
        }

        TreeNode<Key, T>* current = root;
        TreeNode<Key, T>* parent = nullptr;

        while (current != nullptr) {
            parent = current;
            if (key < current->data.first) {
                current = current->left;
            } else if (key > current->data.first) {
                current = current->right;
            } else {
                // 键已存在，更新值
                current->data.second = value;
                return;
            }
        }

        if (key < parent->data.first) {
            parent->left = new TreeNode<Key, T>(key, value, parent);
        } else {
            parent->right = new TreeNode<Key, T>(key, value, parent);
        }
    }

    // 查找元素，返回指向节点的指针
    TreeNode<Key, T>* find(const Key& key) const {
        TreeNode<Key, T>* current = root;
        while (current != nullptr) {
            if (key < current->data.first) {
                current = current->left;
            } else if (key > current->data.first) {
                current = current->right;
            } else {
                return current;
            }
        }
        return nullptr;
    }

    // 删除元素
    void erase(const Key& key) {
        TreeNode<Key, T>* node = find(key);
        if (node == nullptr) return; // 没有找到要删除的节点

        // 节点有两个子节点
        if (node->left != nullptr && node->right != nullptr) {
            // 找到中序后继
            TreeNode<Key, T>* successor = minimum(node->right);
            node->data = successor->data; // 替换数据
            node = successor; // 将要删除的节点指向后继节点
        }

        // 节点有一个或没有子节点
        TreeNode<Key, T>* child = (node->left) ? node->left : node->right;
        if (child != nullptr) {
            child->parent = node->parent;
        }

        if (node->parent == nullptr) {
            root = child;
        } else if (node == node->parent->left) {
            node->parent->left = child;
        } else {
            node->parent->right = child;
        }

        delete node;
    }

    // 清空所有节点
    void clear() {
        clear(root);
        root = nullptr;
    }

    // 获取迭代器
    class Iterator {
    public:
        Iterator(TreeNode<Key, T>* node = nullptr) : current(node) {}

        std::pair<const Key, T>& operator*() const {
            return current->data;
        }

        std::pair<const Key, T>* operator->() const {
            return &(current->data);
        }

        // 前置递增
        Iterator& operator++() {
            current = successor(current);
            return *this;
        }

        // 后置递增
        Iterator operator++(int) {
            Iterator temp = *this;
            current = successor(current);
            return temp;
        }

        bool operator==(const Iterator& other) const {
            return current == other.current;
        }

        bool operator!=(const Iterator& other) const {
            return current != other.current;
        }

    private:
        TreeNode<Key, T>* current;

        TreeNode<Key, T>* minimum(TreeNode<Key, T>* node) const {
            while (node->left != nullptr) {
                node = node->left;
            }
            return node;
        }

        TreeNode<Key, T>* successor(TreeNode<Key, T>* node) const {
            if (node->right != nullptr) {
                return minimum(node->right);
            }

            TreeNode<Key, T>* p = node->parent;
            while (p != nullptr && node == p->right) {
                node = p;
                p = p->parent;
            }
            return p;
        }
    };

    Iterator begin() const {
        return Iterator(minimum(root));
    }

    Iterator end() const {
        return Iterator(nullptr);
    }

private:
    TreeNode<Key, T>* root;

    // 删除树中的所有节点
    void clear(TreeNode<Key, T>* node) {
        if (node == nullptr) return;
        clear(node->left);
        clear(node->right);
        delete node;
    }

    // 找到最小的节点
    TreeNode<Key, T>* minimum(TreeNode<Key, T>* node) const {
        if (node == nullptr) return nullptr;
        while (node->left != nullptr) {
            node = node->left;
        }
        return node;
    }

    // 找到最大的节点
    TreeNode<Key, T>* maximum(TreeNode<Key, T>* node) const {
        if (node == nullptr) return nullptr;
        while (node->right != nullptr) {
            node = node->right;
        }
        return node;
    }
};
```



### 2.3 解释



1. **TreeNode 结构**：

   - `data`: 存储键值对 `std::pair<Key, T>`。
   - `left` 和 `right`: 指向左子节点和右子节点。
   - `parent`: 指向父节点，便于迭代器中查找后继节点。

2. **MyMap 类**：

   - 构造与析构

     ：

     - 构造函数初始化根节点为空。
     - 析构函数调用 `clear` 释放所有节点内存。

   - 插入 (`insert`)

     ：

     - 从根节点开始，根据键的大小确定插入左子树还是右子树。
     - 如果键已存在，更新对应的值。

   - 查找 (`find`)

     ：

     - 按照键的大小在树中查找对应的节点。

   - 删除 (`erase`)

     ：

     - 查找到目标节点。
     - 如果节点有两个子节点，找到中序后继节点并替换当前节点的数据，然后删除后继节点。
     - 如果节点有一个或没有子节点，直接替换节点指针并删除节点。

   - 清空 (`clear`)

     ：

     - 使用递归方式删除所有节点。

   - 迭代器

     ：

     - 定义了嵌套的 `Iterator` 类，支持前置和后置递增操作。
     - 迭代器通过中序遍历实现，保证键的顺序性。
     - `begin()` 返回最小的节点，`end()` 返回 `nullptr`。



## 3. 使用示例



下面提供一个使用 `MyMap` 的示例，展示插入、查找、删除和迭代操作。



```cpp
int main() {
    MyMap<std::string, int> myMap;

    // 插入元素
    myMap.insert("apple", 3);
    myMap.insert("banana", 5);
    myMap.insert("orange", 2);
    myMap.insert("grape", 7);
    myMap.insert("cherry", 4);

    // 使用迭代器遍历元素（按键的字母顺序）
    std::cout << "Map contents (in-order):\n";
    for(auto it = myMap.begin(); it != myMap.end(); ++it) {
        std::cout << it->first << " => " << it->second << "\n";
    }

    // 查找元素
    std::string keyToFind = "banana";
    auto node = myMap.find(keyToFind);
    if(node != nullptr) {
        std::cout << "\nFound " << keyToFind << " with value: " << node->data.second << "\n";
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



### 输出结果



```makefile
Map contents (in-order):
apple => 3
banana => 5
cherry => 4
grape => 7
orange => 2

Found banana with value: 5

After erasing apple and cherry:
banana => 5
grape => 7
orange => 2
```



## 4. 迭代器的详细实现



为了支持迭代器的正常使用，`Iterator` 类实现了以下功能：



- **解引用操作符 (`operator\*` 和 `operator->`)**：
  - 允许访问键值对，如 `it->first` 和 `it->second`。
- **递增操作符 (`operator++`)**：
  - 前置递增（`++it`）和后置递增（`it++`）用于移动到下一个元素。
  - 通过查找当前节点的中序后继节点实现。
- **比较操作符 (`operator==` 和 `operator!=`)**：
  - 判断两个迭代器是否指向同一个节点。



### 中序后继节点的查找



迭代器使用中序遍历来确保键的有序性。计算后继节点的步骤如下：



1. **当前节点有右子树**：
   - 后继节点是右子树中的最小节点。
2. **当前节点没有右子树**：
   - 向上查找，直到找到一个节点是其父节点的左子树，此时父节点即为后继节点。



如果没有后继节点（即当前节点是最大的节点），则返回 `nullptr`，表示迭代器到达 `end()`。



## 5. 扩展功能



上述实现是一个基本的 `Map`，还可以根据需要扩展更多功能，例如：



- **支持 `const` 迭代器**：
  - 定义 `const_iterator`，确保在只读操作时数据不被修改。
- **实现平衡树**：
  - 为了提高性能，可以实现红黑树、AVL 树等自平衡二叉搜索树，保证操作的时间复杂度为 O(log n)。
- **添加更多成员函数**：
  - 如 `operator[]`、`count`、`lower_bound`、`upper_bound` 等，增加容器的功能性。
- **异常处理**：
  - 增加对异常情况的处理，例如在删除不存在的键时抛出异常等。
- **迭代器的逆向遍历**：
  - 实现双向迭代器，支持逆序遍历（`rbegin()` 和 `rend()`）。

# AVL树

AVL树（Adelson-Velsky and Landis树）是一种自平衡的二叉搜索树（BST），它在插入和删除操作后通过旋转来保持树的平衡，从而确保基本操作（如查找、插入和删除）的时间复杂度保持在O(log n)。使用AVL树来实现`map`（键值对映射）是一个高效的选择，特别适合需要频繁查找、插入和删除操作的场景。



## 1. 模板化AVL树节点



首先，我们需要将`AVLNode`结构模板化，使其能够处理不同类型的键和值。我们假设键类型`KeyType`支持`operator<`进行比较，因为AVL树需要对键进行排序以维护其性质。

首先，我们定义AVL树的节点。每个节点包含一个键（`key`）、一个值（`value`）、节点高度（`height`），以及指向左子节点和右子节点的指针。

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm> // 用于 std::max
#include <functional> // 用于 std::function

// 模板化的AVL树节点结构
template <typename KeyType, typename ValueType>
struct AVLNode {
    KeyType key;
    ValueType value;
    int height;
    AVLNode* left;
    AVLNode* right;

    AVLNode(const KeyType& k, const ValueType& val)
        : key(k), value(val), height(1), left(nullptr), right(nullptr) {}
};
```



> **说明**：
>
> - `KeyType`：键的类型，需要支持比较操作（如 `operator<`）。
> - `ValueType`：值的类型，可以是任何类型。



## 2. 辅助函数的模板化



辅助函数同样需要模板化，以适应不同的`AVLNode`类型。



### 获取节点高度

获取节点的高度。如果节点为空，则高度为0。

```cpp
template <typename KeyType, typename ValueType>
int getHeight(AVLNode<KeyType, ValueType>* node) {
    if (node == nullptr)
        return 0;
    return node->height;
}
```



### 获取平衡因子

平衡因子（Balance Factor）是左子树高度减去右子树高度。

```cpp
template <typename KeyType, typename ValueType>
int getBalance(AVLNode<KeyType, ValueType>* node) {
    if (node == nullptr)
        return 0;
    return getHeight(node->left) - getHeight(node->right);
}
```



### 右旋转

右旋转用于处理左子树过高的情况（例如，左左情况）。

```markdown
    y                             x
   / \                           / \
  x   T3            ==>          z   y
 / \                               / \
z   T2                            T2  T3
```

实现

```cpp
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* rightRotate(AVLNode<KeyType, ValueType>* y) {
    AVLNode<KeyType, ValueType>* x = y->left;
    AVLNode<KeyType, ValueType>* T2 = x->right;

    // 执行旋转
    x->right = y;
    y->left = T2;

    // 更新高度
    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;
    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;

    // 返回新的根
    return x;
}
```



### 左旋转

左旋转用于处理右子树过高的情况（例如，右右情况）。



```markdown
    x                             y
   / \                           / \
  T1  y           ==>            x   z
     / \                       / \
    T2  z                     T1  T2
```

具体实现

```cpp
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* leftRotate(AVLNode<KeyType, ValueType>* x) {
    AVLNode<KeyType, ValueType>* y = x->right;
    AVLNode<KeyType, ValueType>* T2 = y->left;

    // 执行旋转
    y->left = x;
    x->right = T2;

    // 更新高度
    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;
    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;

    // 返回新的根
    return y;
}
```



## 3. AVL树的核心操作模板化



### 插入节点



插入操作遵循标准的二叉搜索树插入方式，然后通过旋转保持树的平衡。



```cpp
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* insertNode(AVLNode<KeyType, ValueType>* node, const KeyType& key, const ValueType& value) {
    // 1. 执行标准的BST插入
    if (node == nullptr)
        return new AVLNode<KeyType, ValueType>(key, value);

    if (key < node->key)
        node->left = insertNode(node->left, key, value);
    else if (key > node->key)
        node->right = insertNode(node->right, key, value);
    else {
        // 如果键已经存在，更新其值
        node->value = value;
        return node;
    }

    // 2. 更新节点高度
    node->height = 1 + std::max(getHeight(node->left), getHeight(node->right));

    // 3. 获取平衡因子
    int balance = getBalance(node);

    // 4. 根据平衡因子进行旋转

    // 左左情况
    if (balance > 1 && key < node->left->key)
        return rightRotate(node);

    // 右右情况
    if (balance < -1 && key > node->right->key)
        return leftRotate(node);

    // 左右情况
    if (balance > 1 && key > node->left->key) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
    }

    // 右左情况
    if (balance < -1 && key < node->right->key) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
    }

    return node;
}
```



### 查找节点

按键查找节点，返回对应的值。如果键不存在，返回`nullptr`。

```cpp
template <typename KeyType, typename ValueType>
ValueType* searchNode(AVLNode<KeyType, ValueType>* node, const KeyType& key) {
    if (node == nullptr)
        return nullptr;

    if (key == node->key)
        return &(node->value);
    else if (key < node->key)
        return searchNode(node->left, key);
    else
        return searchNode(node->right, key);
}
```



### 获取最小值节点

用于删除节点时找到中序后继。

```cpp
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* getMinValueNode(AVLNode<KeyType, ValueType>* node) {
    AVLNode<KeyType, ValueType>* current = node;
    while (current->left != nullptr)
        current = current->left;
    return current;
}
```



### 删除节点

删除操作分为三种情况：删除节点是叶子节点、有一个子节点或有两个子节点。删除后，通过旋转保持树的平衡。

```cpp
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* deleteNode(AVLNode<KeyType, ValueType>* root, const KeyType& key) {
    // 1. 执行标准的BST删除
    if (root == nullptr)
        return root;

    if (key < root->key)
        root->left = deleteNode(root->left, key);
    else if (key > root->key)
        root->right = deleteNode(root->right, key);
    else {
        // 节点有一个或没有子节点
        if ((root->left == nullptr) || (root->right == nullptr)) {
            AVLNode<KeyType, ValueType>* temp = root->left ? root->left : root->right;

            // 没有子节点
            if (temp == nullptr) {
                temp = root;
                root = nullptr;
            }
            else // 一个子节点
                *root = *temp; // 复制内容

            delete temp;
        }
        else {
            // 节点有两个子节点，获取中序后继
            AVLNode<KeyType, ValueType>* temp = getMinValueNode(root->right);
            // 复制中序后继的内容到此节点
            root->key = temp->key;
            root->value = temp->value;
            // 删除中序后继
            root->right = deleteNode(root->right, temp->key);
        }
    }

    // 如果树只有一个节点
    if (root == nullptr)
        return root;

    // 2. 更新节点高度
    root->height = 1 + std::max(getHeight(root->left), getHeight(root->right));

    // 3. 获取平衡因子
    int balance = getBalance(root);

    // 4. 根据平衡因子进行旋转

    // 左左情况
    if (balance > 1 && getBalance(root->left) >= 0)
        return rightRotate(root);

    // 左右情况
    if (balance > 1 && getBalance(root->left) < 0) {
        root->left = leftRotate(root->left);
        return rightRotate(root);
    }

    // 右右情况
    if (balance < -1 && getBalance(root->right) <= 0)
        return leftRotate(root);

    // 右左情况
    if (balance < -1 && getBalance(root->right) > 0) {
        root->right = rightRotate(root->right);
        return leftRotate(root);
    }

    return root;
}
```



## 4. 模板化的AVLMap类



现在，我们将所有模板化的函数集成到一个模板类`AVLMap`中。这个类将提供如下功能：



- `put(const KeyType& key, const ValueType& value)`：插入或更新键值对。
- `get(const KeyType& key)`：查找键对应的值，返回指向值的指针，如果键不存在则返回`nullptr`。
- `remove(const KeyType& key)`：删除指定键的键值对。
- `inorderTraversal()`：中序遍历，返回有序的键值对列表。



```cpp
template <typename KeyType, typename ValueType>
class AVLMap {
private:
    AVLNode<KeyType, ValueType>* root;

    // 中序遍历辅助函数
    void inorderHelper(AVLNode<KeyType, ValueType>* node, std::vector<std::pair<KeyType, ValueType>>& res) const {
        if (node != nullptr) {
            inorderHelper(node->left, res);
            res.emplace_back(node->key, node->value);
            inorderHelper(node->right, res);
        }
    }

public:
    AVLMap() : root(nullptr) {}

    // 插入或更新键值对
    void put(const KeyType& key, const ValueType& value) {
        root = insertNode(root, key, value);
    }

    // 查找值，返回指向值的指针，如果键不存在则返回nullptr
    ValueType* get(const KeyType& key) {
        return searchNode(root, key);
    }

    // 删除键值对
    void remove(const KeyType& key) {
        root = deleteNode(root, key);
    }

    // 中序遍历，返回有序的键值对
    std::vector<std::pair<KeyType, ValueType>> inorderTraversal() const {
        std::vector<std::pair<KeyType, ValueType>> res;
        inorderHelper(root, res);
        return res;
    }

    // 析构函数，释放所有节点的内存
    ~AVLMap() {
        // 使用后序遍历释放节点
        std::function<void(AVLNode<KeyType, ValueType>*)> destroy = [&](AVLNode<KeyType, ValueType>* node) {
            if (node) {
                destroy(node->left);
                destroy(node->right);
                delete node;
            }
        };
        destroy(root);
    }
};
```



> **说明**：
>
> - 模板参数
>
>   ：
>
>   - `KeyType`：键的类型，需要支持`operator<`进行比较。
>   - `ValueType`：值的类型，可以是任意类型。
>
> - 内存管理
>
>   ：
>
>   - 析构函数使用后序遍历释放所有动态分配的节点，防止内存泄漏。
>
> - 异常安全
>
>   ：
>
>   - 当前实现没有处理异常情况。如果需要更高的异常安全性，可以进一步增强代码，例如在插入过程中捕获异常并回滚操作。



## 5. 使用示例



下面的示例展示了如何使用模板化的`AVLMap`类，使用不同类型的键和值。



### 示例 1：使用`int`作为键，`std::string`作为值



```cpp
#include <iostream>
#include <string>
#include <vector>

// 假设上面的AVLNode结构、辅助函数和AVLMap类已经定义

int main() {
    AVLMap<int, std::string> avlMap;

    // 插入键值对
    avlMap.put(10, "十");
    avlMap.put(20, "二十");
    avlMap.put(30, "三十");
    avlMap.put(40, "四十");
    avlMap.put(50, "五十");
    avlMap.put(25, "二十五");

    // 中序遍历
    std::vector<std::pair<int, std::string>> traversal = avlMap.inorderTraversal();
    std::cout << "中序遍历: ";
    for (const auto& pair : traversal) {
        std::cout << "(" << pair.first << ", \"" << pair.second << "\") ";
    }
    std::cout << std::endl;

    // 查找键
    std::string* val = avlMap.get(20);
    if (val)
        std::cout << "获取键20的值: " << *val << std::endl;
    else
        std::cout << "键20不存在。" << std::endl;

    val = avlMap.get(25);
    if (val)
        std::cout << "获取键25的值: " << *val << std::endl;
    else
        std::cout << "键25不存在。" << std::endl;

    val = avlMap.get(60);
    if (val)
        std::cout << "获取键60的值: " << *val << std::endl;
    else
        std::cout << "键60不存在。" << std::endl;

    // 删除键20
    avlMap.remove(20);
    std::cout << "删除键20后，中序遍历: ";
    traversal = avlMap.inorderTraversal();
    for (const auto& pair : traversal) {
        std::cout << "(" << pair.first << ", \"" << pair.second << "\") ";
    }
    std::cout << std::endl;

    return 0;
}
```



**输出**:



```bash
中序遍历: (10, "十") (20, "二十") (25, "二十五") (30, "三十") (40, "四十") (50, "五十") 
获取键20的值: 二十
获取键25的值: 二十五
键60不存在。
删除键20后，中序遍历: (10, "十") (25, "二十五") (30, "三十") (40, "四十") (50, "五十") 
```



### 示例 2：使用`std::string`作为键，`double`作为值



```cpp
#include <iostream>
#include <string>
#include <vector>

// 假设上面的AVLNode结构、辅助函数和AVLMap类已经定义

int main() {
    AVLMap<std::string, double> avlMap;

    // 插入键值对
    avlMap.put("apple", 1.99);
    avlMap.put("banana", 0.99);
    avlMap.put("cherry", 2.99);
    avlMap.put("date", 3.49);
    avlMap.put("elderberry", 5.99);
    avlMap.put("fig", 2.49);

    // 中序遍历
    std::vector<std::pair<std::string, double>> traversal = avlMap.inorderTraversal();
    std::cout << "中序遍历: ";
    for (const auto& pair : traversal) {
        std::cout << "(\"" << pair.first << "\", " << pair.second << ") ";
    }
    std::cout << std::endl;

    // 查找键
    double* val = avlMap.get("banana");
    if (val)
        std::cout << "获取键\"banana\"的值: " << *val << std::endl;
    else
        std::cout << "键\"banana\"不存在。" << std::endl;

    val = avlMap.get("fig");
    if (val)
        std::cout << "获取键\"fig\"的值: " << *val << std::endl;
    else
        std::cout << "键\"fig\"不存在。" << std::endl;

    val = avlMap.get("grape");
    if (val)
        std::cout << "获取键\"grape\"的值: " << *val << std::endl;
    else
        std::cout << "键\"grape\"不存在。" << std::endl;

    // 删除键"banana"
    avlMap.remove("banana");
    std::cout << "删除键\"banana\"后，中序遍历: ";
    traversal = avlMap.inorderTraversal();
    for (const auto& pair : traversal) {
        std::cout << "(\"" << pair.first << "\", " << pair.second << ") ";
    }
    std::cout << std::endl;

    return 0;
}
```



**输出**:



```bash
中序遍历: ("apple", 1.99) ("banana", 0.99) ("cherry", 2.99) ("date", 3.49) ("elderberry", 5.99) ("fig", 2.49) 
获取键"banana"的值: 0.99
获取键"fig"的值: 2.49
键"grape"不存在。
删除键"banana"后，中序遍历: ("apple", 1.99) ("cherry", 2.99) ("date", 3.49) ("elderberry", 5.99) ("fig", 2.49) 
```



## 6. 完整的通用代码



以下是模板化的`AVLMap`的完整实现代码，包括所有辅助函数和类定义：



```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>   // 用于 std::max
#include <functional>  // 用于 std::function

// 模板化的AVL树节点结构
template <typename KeyType, typename ValueType>
struct AVLNode {
    KeyType key;
    ValueType value;
    int height;
    AVLNode* left;
    AVLNode* right;

    AVLNode(const KeyType& k, const ValueType& val)
        : key(k), value(val), height(1), left(nullptr), right(nullptr) {}
};

// 获取节点高度
template <typename KeyType, typename ValueType>
int getHeight(AVLNode<KeyType, ValueType>* node) {
    if (node == nullptr)
        return 0;
    return node->height;
}

// 获取平衡因子
template <typename KeyType, typename ValueType>
int getBalance(AVLNode<KeyType, ValueType>* node) {
    if (node == nullptr)
        return 0;
    return getHeight(node->left) - getHeight(node->right);
}

// 右旋转
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* rightRotate(AVLNode<KeyType, ValueType>* y) {
    AVLNode<KeyType, ValueType>* x = y->left;
    AVLNode<KeyType, ValueType>* T2 = x->right;

    // 执行旋转
    x->right = y;
    y->left = T2;

    // 更新高度
    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;
    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;

    // 返回新的根
    return x;
}

// 左旋转
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* leftRotate(AVLNode<KeyType, ValueType>* x) {
    AVLNode<KeyType, ValueType>* y = x->right;
    AVLNode<KeyType, ValueType>* T2 = y->left;

    // 执行旋转
    y->left = x;
    x->right = T2;

    // 更新高度
    x->height = std::max(getHeight(x->left), getHeight(x->right)) + 1;
    y->height = std::max(getHeight(y->left), getHeight(y->right)) + 1;

    // 返回新的根
    return y;
}

// 插入节点
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* insertNode(AVLNode<KeyType, ValueType>* node, const KeyType& key, const ValueType& value) {
    // 1. 执行标准的BST插入
    if (node == nullptr)
        return new AVLNode<KeyType, ValueType>(key, value);

    if (key < node->key)
        node->left = insertNode(node->left, key, value);
    else if (key > node->key)
        node->right = insertNode(node->right, key, value);
    else {
        // 如果键已经存在，更新其值
        node->value = value;
        return node;
    }

    // 2. 更新节点高度
    node->height = 1 + std::max(getHeight(node->left), getHeight(node->right));

    // 3. 获取平衡因子
    int balance = getBalance(node);

    // 4. 根据平衡因子进行旋转

    // 左左情况
    if (balance > 1 && key < node->left->key)
        return rightRotate(node);

    // 右右情况
    if (balance < -1 && key > node->right->key)
        return leftRotate(node);

    // 左右情况
    if (balance > 1 && key > node->left->key) {
        node->left = leftRotate(node->left);
        return rightRotate(node);
    }

    // 右左情况
    if (balance < -1 && key < node->right->key) {
        node->right = rightRotate(node->right);
        return leftRotate(node);
    }

    return node;
}

// 查找节点
template <typename KeyType, typename ValueType>
ValueType* searchNode(AVLNode<KeyType, ValueType>* node, const KeyType& key) {
    if (node == nullptr)
        return nullptr;

    if (key == node->key)
        return &(node->value);
    else if (key < node->key)
        return searchNode(node->left, key);
    else
        return searchNode(node->right, key);
}

// 获取最小值节点
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* getMinValueNode(AVLNode<KeyType, ValueType>* node) {
    AVLNode<KeyType, ValueType>* current = node;
    while (current->left != nullptr)
        current = current->left;
    return current;
}

// 删除节点
template <typename KeyType, typename ValueType>
AVLNode<KeyType, ValueType>* deleteNode(AVLNode<KeyType, ValueType>* root, const KeyType& key) {
    // 1. 执行标准的BST删除
    if (root == nullptr)
        return root;

    if (key < root->key)
        root->left = deleteNode(root->left, key);
    else if (key > root->key)
        root->right = deleteNode(root->right, key);
    else {
        // 节点有一个或没有子节点
        if ((root->left == nullptr) || (root->right == nullptr)) {
            AVLNode<KeyType, ValueType>* temp = root->left ? root->left : root->right;

            // 没有子节点
            if (temp == nullptr) {
                temp = root;
                root = nullptr;
            }
            else // 一个子节点
                *root = *temp; // 复制内容

            delete temp;
        }
        else {
            // 节点有两个子节点，获取中序后继
            AVLNode<KeyType, ValueType>* temp = getMinValueNode(root->right);
            // 复制中序后继的内容到此节点
            root->key = temp->key;
            root->value = temp->value;
            // 删除中序后继
            root->right = deleteNode(root->right, temp->key);
        }
    }

    // 如果树只有一个节点
    if (root == nullptr)
        return root;

    // 2. 更新节点高度
    root->height = 1 + std::max(getHeight(root->left), getHeight(root->right));

    // 3. 获取平衡因子
    int balance = getBalance(root);

    // 4. 根据平衡因子进行旋转

    // 左左情况
    if (balance > 1 && getBalance(root->left) >= 0)
        return rightRotate(root);

    // 左右情况
    if (balance > 1 && getBalance(root->left) < 0) {
        root->left = leftRotate(root->left);
        return rightRotate(root);
    }

    // 右右情况
    if (balance < -1 && getBalance(root->right) <= 0)
        return leftRotate(root);

    // 右左情况
    if (balance < -1 && getBalance(root->right) > 0) {
        root->right = rightRotate(root->right);
        return leftRotate(root);
    }

    return root;
}

// 模板化的AVLMap类
template <typename KeyType, typename ValueType>
class AVLMap {
private:
    AVLNode<KeyType, ValueType>* root;

    // 中序遍历辅助函数
    void inorderHelper(AVLNode<KeyType, ValueType>* node, std::vector<std::pair<KeyType, ValueType>>& res) const {
        if (node != nullptr) {
            inorderHelper(node->left, res);
            res.emplace_back(node->key, node->value);
            inorderHelper(node->right, res);
        }
    }

public:
    AVLMap() : root(nullptr) {}

    // 插入或更新键值对
    void put(const KeyType& key, const ValueType& value) {
        root = insertNode(root, key, value);
    }

    // 查找值，返回指向值的指针，如果键不存在则返回nullptr
    ValueType* get(const KeyType& key) {
        return searchNode(root, key);
    }

    // 删除键值对
    void remove(const KeyType& key) {
        root = deleteNode(root, key);
    }

    // 中序遍历，返回有序的键值对
    std::vector<std::pair<KeyType, ValueType>> inorderTraversal() const {
        std::vector<std::pair<KeyType, ValueType>> res;
        inorderHelper(root, res);
        return res;
    }

    // 析构函数，释放所有节点的内存
    ~AVLMap() {
        // 使用后序遍历释放节点
        std::function<void(AVLNode<KeyType, ValueType>*)> destroy = [&](AVLNode<KeyType, ValueType>* node) {
            if (node) {
                destroy(node->left);
                destroy(node->right);
                delete node;
            }
        };
        destroy(root);
    }
};

// 示例主函数
int main() {
    // 示例 1：int 键，std::string 值
    std::cout << "示例 1：int 键，std::string 值\n";
    AVLMap<int, std::string> avlMap1;

    // 插入键值对
    avlMap1.put(10, "十");
    avlMap1.put(20, "二十");
    avlMap1.put(30, "三十");
    avlMap1.put(40, "四十");
    avlMap1.put(50, "五十");
    avlMap1.put(25, "二十五");

    // 中序遍历
    std::vector<std::pair<int, std::string>> traversal1 = avlMap1.inorderTraversal();
    std::cout << "中序遍历: ";
    for (const auto& pair : traversal1) {
        std::cout << "(" << pair.first << ", \"" << pair.second << "\") ";
    }
    std::cout << std::endl;

    // 查找键
    std::string* val1 = avlMap1.get(20);
    if (val1)
        std::cout << "获取键20的值: " << *val1 << std::endl;
    else
        std::cout << "键20不存在。" << std::endl;

    val1 = avlMap1.get(25);
    if (val1)
        std::cout << "获取键25的值: " << *val1 << std::endl;
    else
        std::cout << "键25不存在。" << std::endl;

    val1 = avlMap1.get(60);
    if (val1)
        std::cout << "获取键60的值: " << *val1 << std::endl;
    else
        std::cout << "键60不存在。" << std::endl;

    // 删除键20
    avlMap1.remove(20);
    std::cout << "删除键20后，中序遍历: ";
    traversal1 = avlMap1.inorderTraversal();
    for (const auto& pair : traversal1) {
        std::cout << "(" << pair.first << ", \"" << pair.second << "\") ";
    }
    std::cout << std::endl;

    std::cout << "\n-----------------------------\n";

    // 示例 2：std::string 键，double 值
    std::cout << "示例 2：std::string 键，double 值\n";
    AVLMap<std::string, double> avlMap2;

    // 插入键值对
    avlMap2.put("apple", 1.99);
    avlMap2.put("banana", 0.99);
    avlMap2.put("cherry", 2.99);
    avlMap2.put("date", 3.49);
    avlMap2.put("elderberry", 5.99);
    avlMap2.put("fig", 2.49);

    // 中序遍历
    std::vector<std::pair<std::string, double>> traversal2 = avlMap2.inorderTraversal();
    std::cout << "中序遍历: ";
    for (const auto& pair : traversal2) {
        std::cout << "(\"" << pair.first << "\", " << pair.second << ") ";
    }
    std::cout << std::endl;

    // 查找键
    double* val2 = avlMap2.get("banana");
    if (val2)
        std::cout << "获取键\"banana\"的值: " << *val2 << std::endl;
    else
        std::cout << "键\"banana\"不存在。" << std::endl;

    val2 = avlMap2.get("fig");
    if (val2)
        std::cout << "获取键\"fig\"的值: " << *val2 << std::endl;
    else
        std::cout << "键\"fig\"不存在。" << std::endl;

    val2 = avlMap2.get("grape");
    if (val2)
        std::cout << "获取键\"grape\"的值: " << *val2 << std::endl;
    else
        std::cout << "键\"grape\"不存在。" << std::endl;

    // 删除键"banana"
    avlMap2.remove("banana");
    std::cout << "删除键\"banana\"后，中序遍历: ";
    traversal2 = avlMap2.inorderTraversal();
    for (const auto& pair : traversal2) {
        std::cout << "(\"" << pair.first << "\", " << pair.second << ") ";
    }
    std::cout << std::endl;

    return 0;
}
```

## 说明



1. **平衡维护**：在每次插入或删除后，都会更新节点的高度并计算平衡因子。如果某个节点的平衡因子超出了`[-1, 1]`范围，就需要通过旋转来重新平衡树。
2. **查找操作**：由于AVL树的高度被保持在`O(log n)`，查找操作的时间复杂度为`O(log n)`。
3. **更新操作**：如果插入的键已经存在，则更新其对应的值。
4. **遍历操作**：中序遍历可以按键的顺序遍历所有键值对。
5. **内存管理**：确保在析构函数中正确释放所有动态分配的内存，避免内存泄漏。
6. **泛型支持（可选）**：为了使`AVLMap`更加通用，可以将其模板化，以支持不同类型的键和值。例如：





**编译与运行**：



假设保存为 `avlmap.cpp`，使用以下命令编译并运行：



```bash
g++ -std=c++11 -o avlmap avlmap.cpp
./avlmap
```



**预期输出**：



```cpp
示例 1：int 键，std::string 值
中序遍历: (10, "十") (20, "二十") (25, "二十五") (30, "三十") (40, "四十") (50, "五十") 
获取键20的值: 二十
获取键25的值: 二十五
键60不存在。
删除键20后，中序遍历: (10, "十") (25, "二十五") (30, "三十") (40, "四十") (50, "五十") 

-----------------------------
示例 2：std::string 键，double 值
中序遍历: ("apple", 1.99) ("banana", 0.99) ("cherry", 2.99) ("date", 3.49) ("elderberry", 5.99) ("fig", 2.49) 
获取键"banana"的值: 0.99
获取键"fig"的值: 2.49
键"grape"不存在。
删除键"banana"后，中序遍历: ("apple", 1.99) ("cherry", 2.99) ("date", 3.49) ("elderberry", 5.99) ("fig", 2.49) 
```



## 7. 注意事项与扩展



### 1. 键类型的要求



为了使`AVLMap`正常工作，键类型`KeyType`必须支持以下操作：



- **比较操作**：必须定义`operator<`，因为AVL树依赖于它来维护排序。如果使用自定义类型作为键，请确保定义了`operator<`。

  ```cpp
  struct CustomKey {
      int id;
      std::string name;
  
      bool operator<(const CustomKey& other) const {
          if (id != other.id)
              return id < other.id;
          return name < other.name;
      }
  };
  ```



### 2. 泛型支持与约束



在C++20之前，模板并不支持在模板参数上强制施加约束（需要依赖文档和用户理解）。从C++20起，可以使用**概念（Concepts）**来施加约束。



```cpp
#include <concepts>

template <typename KeyType, typename ValueType>
requires std::totally_ordered<KeyType>
class AVLMap {
    // 类定义
};
```



这样，编译器会在实例化模板时检查`KeyType`是否满足`std::totally_ordered`，即是否支持所有必要的比较操作。



### 3. 性能优化



- **内存管理**：当前实现使用递归进行插入和删除，如果树非常深，可能会导致栈溢出。可以考虑使用迭代方法或优化递归深度。
- **缓存友好**：使用自适应数据结构（如缓存友好的节点布局）可以提升性能。
- **多线程支持**：当前实现不是线程安全的。如果需要在多线程环境中使用，需要添加适当的同步机制。



### 4. 额外功能



根据需求，你可以为`AVLMap`添加更多功能：



- **迭代器**：实现输入迭代器、中序遍历迭代器，以便支持范围基（range-based）`for`循环。
- **查找最小/最大键**：提供方法`findMin()`和`findMax()`。
- **前驱/后继查找**：在树中查找给定键的前驱和后继节点。
- **支持不同的平衡因子策略**：例如，允许用户指定自定义的平衡策略。



### 5. 与标准库的比较



虽然自己实现`AVLMap`是一项很好的学习练习，但在实际生产环境中，建议使用C++标准库中已经高度优化和测试过的容器，如`std::map`（通常实现为红黑树）、`std::unordered_map`（哈希表）等。



```cpp
#include <map>

// 使用 std::map
int main() {
    std::map<int, std::string> stdMap;

    // 插入键值对
    stdMap[10] = "十";
    stdMap[20] = "二十";
    stdMap[30] = "三十";

    // 遍历
    for (const auto& pair : stdMap) {
        std::cout << "(" << pair.first << ", \"" << pair.second << "\") ";
    }
    std::cout << std::endl;

    return 0;
}
```



`std::map`提供了与`AVLMap`类似的功能，并且经过了高度优化，适用于大多数应用场景。



# 红黑树

红黑树（**Red-Black Tree**）是一种自平衡的二叉搜索树，它通过对节点进行颜色标记（红色或黑色）并遵循特定的规则来保证树的平衡性，从而确保基本操作（如查找、插入、删除）的时间复杂度为 **O(log n)**。红黑树广泛应用于计算机科学中，例如在实现关联容器（如 `std::map`、`std::set`）时常用到。

## 1. 红黑树的五大性质



红黑树通过以下 **五大性质** 维持其平衡性：



1. **节点颜色**：每个节点要么是**红色**，要么是**黑色**。
2. **根节点颜色**：根节点是**黑色**。
3. **叶子节点颜色**：所有叶子节点（NIL 节点，即空节点）都是**黑色**的。这里的叶子节点不存储实际数据，仅作为树的终端。
4. **红色节点限制**：如果一个节点是**红色**的，则它的两个子节点都是**黑色**的。也就是说，**红色节点不能有红色的子节点**。
5. **黑色平衡**：从任意节点到其所有后代叶子节点的路径上，**包含相同数量的黑色节点**。这被称为每条路径上的**黑色高度**相同。



### 这些性质的意义



- **性质1** 和 **性质2** 确保节点颜色的基本规则，便于后续操作中进行颜色判断和调整。
- **性质3** 保证了所有实际节点的子节点（NIL 节点）的统一性，简化了操作逻辑。
- **性质4** 防止了连续的红色节点出现，避免导致过度不平衡。
- **性质5** 保证了树的平衡性，使得树的高度始终保持在 **O(log n)** 的范围内，从而确保基本操作的高效性。



这些性质共同作用，使得红黑树在最坏情况下也能保持较好的性能表现。



------



## 2. 红黑树的插入操作



插入操作是红黑树中常见的操作，与标准的二叉搜索树（BST）插入类似，但需要额外的步骤来维护红黑树的性质。



### 2.1 插入步骤概述



插入操作通常分为以下两个主要步骤：



1. **标准二叉搜索树插入**：根据键值比较，将新节点插入到合适的位置，初始颜色为**红色**。
2. **插入后的修正（Insert Fixup）**：通过重新着色和旋转操作，恢复红黑树的五大性质。



### 2.2 插入后的修正（Insert Fixup）



插入一个红色节点可能会破坏红黑树的性质，尤其是**性质4**（红色节点不能连续）。为了修复这些潜在的冲突，需要进行颜色调整和旋转操作。



#### 修正步骤：



插入修正的过程通常遵循以下规则（以下描述假设新插入的节点 **z** 是红色）：



1. **父节点为黑色**：
   - 如果 **z** 的父节点是黑色的，那么插入操作不会破坏红黑树的性质，修正过程结束。
2. **父节点为红色**：
   - **情况1**：**z** 的叔叔节点（即 **z** 的父节点的兄弟节点）也是红色。
     - 将父节点和叔叔节点重新着色为黑色。
     - 将祖父节点重新着色为红色。
     - 将 **z** 指向祖父节点，继续检查上层节点，防止高层的性质被破坏。
   - **情况2**：**z** 的叔叔节点是黑色，且 **z** 是其父节点的右子节点。
     - 对 **z** 的父节点进行左旋转。
     - 将 **z** 指向其新的左子节点（即原父节点）。
   - **情况3**：**z** 的叔叔节点是黑色，且 **z** 是其父节点的左子节点。
     - 将父节点重新着色为黑色。
     - 将祖父节点重新着色为红色。
     - 对祖父节点进行右旋转。



#### 旋转操作的重要性



在修正过程中，**旋转操作**用于调整树的局部结构，使得红黑树的性质得以恢复。这些旋转包括**左旋转**和**右旋转**，在后续章节中将详细介绍。



#### 插入修正的代码实现示例



以下是红黑树插入修正的一个简化版 C++ 实现：



```cpp
template <typename Key, typename Value>
void RedBlackTree<Key, Value>::insertFixUp(RBTreeNode<Key, Value>* z) {
    while (z->parent != nullptr && z->parent->color == RED) {
        if (z->parent == z->parent->parent->left) {
            RBTreeNode<Key, Value>* y = z->parent->parent->right; // 叔叔节点
            if (y != nullptr && y->color == RED) {
                // 情况1：叔叔为红色
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->right) {
                    // 情况2：z为右子节点
                    z = z->parent;
                    leftRotate(z);
                }
                // 情况3：z为左子节点
                z->parent->color = BLACK;
                z->parent->parent->color = RED;
                rightRotate(z->parent->parent);
            }
        } else {
            // 情况对称：父节点是右子节点
            RBTreeNode<Key, Value>* y = z->parent->parent->left; // 叔叔节点
            if (y != nullptr && y->color == RED) {
                // 情况1
                z->parent->color = BLACK;
                y->color = BLACK;
                z->parent->parent->color = RED;
                z = z->parent->parent;
            } else {
                if (z == z->parent->left) {
                    // 情况2
                    z = z->parent;
                    rightRotate(z);
                }
                // 情况3
                z->parent->color = BLACK;
                z->parent->parent->color = RED;
                leftRotate(z->parent->parent);
            }
        }
    }
    root->color = BLACK; // 最终根节点必须为黑色
}
```



------



## 3. 红黑树的删除操作



删除操作同样重要且复杂，因为它可能破坏红黑树的多个性质。与插入类似，删除操作也需要两个主要步骤：



1. **标准二叉搜索树删除**：按照 BST 的规则删除节点。
2. **删除后的修正（Delete Fixup）**：通过重新着色和旋转操作，恢复红黑树的性质。



### 3.1 删除步骤概述



删除操作分为以下步骤：



1. **定位要删除的节点**：
   - 如果要删除的节点 **z** 有两个子节点，则找到其中序后继节点 **y**（即 **z** 的右子树中的最小节点）。
   - 将 **y** 的值复制到 **z**，然后将删除目标转移到 **y**。此时 **y** 至多只有一个子节点。
2. **删除节点**：
   - 若 **y** 只有一个子节点 **x**（可能为 NIL），则用 **x** 替代 **y** 的位置。
   - 记录被删除节点的原颜色 **y_original_color**。
3. **删除修正**（仅当 **y_original_color** 为黑色时）：
   - 因为删除一个黑色节点会影响路径上的黑色数量，需通过多次调整来恢复红黑树的性质。



### 3.2 删除后的修正（Delete Fixup）



删除后的修正较为复杂，涉及多种情况处理。以下为主要的修正步骤和可能遇到的情况：



#### 修正步骤：



1. **初始化**：设 **x** 为替代被删除的节点的位置，**x** 可能为实际节点或 NIL 节点。

2. **循环修正**：

   - 当 **x** 不是根节点，且 **x** 的颜色为**黑色**，进入修正循环。
   - 判断 **x** 是其父节点的左子节点还是右子节点，并相应地设定兄弟节点 **w**。

3. **处理不同情况**：

   **情况1**：**w** 是红色的。

   - 将 **w** 重新着色为黑色。
   - 将 **x** 的父节点重新着色为红色。
   - 对 **x** 的父节点进行左旋转或右旋转，取决于是左子节点还是右子节点。
   - 更新 **w**，继续修正过程。

   **情况2**：**w** 是黑色，且 **w** 的两个子节点都是黑色。

   - 将 **w** 重新着色为红色。
   - 将 **x** 设为其父节点，继续修正。

   **情况3**：**w** 是黑色，且 **w** 的左子节点是红色，右子节点是黑色。

   - 将 **w** 的左子节点重新着色为黑色。
   - 将 **w** 重新着色为红色。
   - 对 **w** 进行右旋转或左旋转，取决于是左子节点还是右子节点。
   - 更新 **w**，进入**情况4**。

   **情况4**：**w** 是黑色，且 **w** 的右子节点是红色。

   - 将 **w** 的颜色设为 **x** 的父节点颜色。
   - 将 **x** 的父节点重新着色为黑色。
   - 将 **w** 的右子节点重新着色为黑色。
   - 对 **x** 的父节点进行左旋转或右旋转，取决于是左子节点还是右子节点。
   - 结束修正。

4. **最终步骤**：将 **x** 设为根节点，并将其颜色设为黑色，确保根节点的颜色为黑色。



#### 删除修正的代码实现示例



由于删除修正涉及较多的情况，以下为一个简化版的红黑树删除修正的 C++ 实现：



```cpp
template <typename Key, typename Value>
void RedBlackTree<Key, Value>::deleteFixUp(RBTreeNode<Key, Value>* x) {
    while (x != root && (x == nullptr || x->color == BLACK)) {
        if (x == x->parent->left) {
            RBTreeNode<Key, Value>* w = x->parent->right; // 兄弟节点
            if (w->color == RED) {
                // 情况1
                w->color = BLACK;
                x->parent->color = RED;
                leftRotate(x->parent);
                w = x->parent->right;
            }
            if ((w->left == nullptr || w->left->color == BLACK) &&
                (w->right == nullptr || w->right->color == BLACK)) {
                // 情况2
                w->color = RED;
                x = x->parent;
            } else {
                if (w->right == nullptr || w->right->color == BLACK) {
                    // 情况3
                    if (w->left != nullptr)
                        w->left->color = BLACK;
                    w->color = RED;
                    rightRotate(w);
                    w = x->parent->right;
                }
                // 情况4
                w->color = x->parent->color;
                x->parent->color = BLACK;
                if (w->right != nullptr)
                    w->right->color = BLACK;
                leftRotate(x->parent);
                x = root; // 修正完成
            }
        } else {
            // 情况对称：x 是右子节点
            RBTreeNode<Key, Value>* w = x->parent->left; // 兄弟节点
            if (w->color == RED) {
                // 情况1
                w->color = BLACK;
                x->parent->color = RED;
                rightRotate(x->parent);
                w = x->parent->left;
            }
            if ((w->right == nullptr || w->right->color == BLACK) &&
                (w->left == nullptr || w->left->color == BLACK)) {
                // 情况2
                w->color = RED;
                x = x->parent;
            } else {
                if (w->left == nullptr || w->left->color == BLACK) {
                    // 情况3
                    if (w->right != nullptr)
                        w->right->color = BLACK;
                    w->color = RED;
                    leftRotate(w);
                    w = x->parent->left;
                }
                // 情况4
                w->color = x->parent->color;
                x->parent->color = BLACK;
                if (w->left != nullptr)
                    w->left->color = BLACK;
                rightRotate(x->parent);
                x = root; // 修正完成
            }
        }
    }
    if (x != nullptr)
        x->color = BLACK;
}
```



------



## 4. 旋转操作详解



**旋转操作**是红黑树中用于重新平衡树的关键操作，包括**左旋转**和**右旋转**。旋转操作通过调整节点的父子关系，改变树的局部结构，从而保持红黑树的性质。



### 4.1 左旋转（Left Rotate）



**左旋转**围绕节点 **x** 进行，其目的是将 **x** 的右子节点 **y** 提升为 **x** 的父节点，**x** 变为 **y** 的左子节点，**y** 的左子节点 **b** 成为 **x** 的右子节点。



**旋转前：**



```css
    x
   / \
  a   y
     / \
    b   c
```



**旋转后：**



```css
      y
     / \
    x   c
   / \
  a   b
```



**左旋转的代码实现：**



```cpp
template <typename Key, typename Value>
void RedBlackTree<Key, Value>::leftRotate(RBTreeNode<Key, Value>* x) {
    RBTreeNode<Key, Value>* y = x->right;
    x->right = y->left;
    if (y->left != nullptr)
        y->left->parent = x;

    y->parent = x->parent;
    if (x->parent == nullptr)
        root = y;
    else if (x == x->parent->left)
        x->parent->left = y;
    else
        x->parent->right = y;

    y->left = x;
    x->parent = y;
}
```



### 4.2 右旋转（Right Rotate）



**右旋转**是 **左旋转** 的镜像操作，围绕节点 **y** 进行，其目的是将 **y** 的左子节点 **x** 提升为 **y** 的父节点，**y** 变为 **x** 的右子节点，**x** 的右子节点 **b** 成为 **y** 的左子节点。



**旋转前：**



```css
      y
     / \
    x   c
   / \
  a   b
```



**旋转后：**



```css
    x
   / \
  a   y
     / \
    b   c
```



**右旋转的代码实现：**



```cpp
template <typename Key, typename Value>
void RedBlackTree<Key, Value>::rightRotate(RBTreeNode<Key, Value>* y) {
    RBTreeNode<Key, Value>* x = y->left;
    y->left = x->right;
    if (x->right != nullptr)
        x->right->parent = y;

    x->parent = y->parent;
    if (y->parent == nullptr)
        root = x;
    else if (y == y->parent->right)
        y->parent->right = x;
    else
        y->parent->left = x;

    x->right = y;
    y->parent = x;
}
```



### 旋转操作的作用



通过旋转操作，可以改变树的高度和形状，确保红黑树的性质在插入和删除后得到维护。旋转不会破坏二叉搜索树的性质，仅改变节点之间的指向关系。



------

## 5.简化版红黑树实现



### 节点结构体



首先，我们定义红黑树节点的结构体：



```cpp
#include <iostream>

enum Color { RED, BLACK };

template <typename Key, typename Value>
struct RBTreeNode {
    Key key;
    Value value;
    Color color;
    RBTreeNode* parent;
    RBTreeNode* left;
    RBTreeNode* right;

    RBTreeNode(Key k, Value v)
        : key(k), value(v), color(RED), parent(nullptr), left(nullptr), right(nullptr) {}
};
```



### 红黑树类



接下来，我们定义红黑树的主要类，包括插入、删除和遍历功能：



```cpp
#include <iostream>

enum Color { RED, BLACK };

template <typename Key, typename Value>
struct RBTreeNode {
    Key key;
    Value value;
    Color color;
    RBTreeNode* parent;
    RBTreeNode* left;
    RBTreeNode* right;

    RBTreeNode(Key k, Value v)
        : key(k), value(v), color(RED), parent(nullptr), left(nullptr), right(nullptr) {}
};

template <typename Key, typename Value>
class RedBlackTree {
private:
    RBTreeNode<Key, Value>* root;

    void leftRotate(RBTreeNode<Key, Value>* x) {
        RBTreeNode<Key, Value>* y = x->right;
        x->right = y->left;
        if (y->left != nullptr)
            y->left->parent = x;

        y->parent = x->parent;
        if (x->parent == nullptr)
            root = y;
        else if (x == x->parent->left)
            x->parent->left = y;
        else
            x->parent->right = y;

        y->left = x;
        x->parent = y;
    }

    void rightRotate(RBTreeNode<Key, Value>* y) {
        RBTreeNode<Key, Value>* x = y->left;
        y->left = x->right;
        if (x->right != nullptr)
            x->right->parent = y;

        x->parent = y->parent;
        if (y->parent == nullptr)
            root = x;
        else if (y == y->parent->right)
            y->parent->right = x;
        else
            y->parent->left = x;

        x->right = y;
        y->parent = x;
    }

    void insertFixUp(RBTreeNode<Key, Value>* z) {
        while (z->parent != nullptr && z->parent->color == RED) {
            if (z->parent == z->parent->parent->left) {
                RBTreeNode<Key, Value>* y = z->parent->parent->right; // 叔叔节点
                if (y != nullptr && y->color == RED) {
                    // 情况1
                    z->parent->color = BLACK;
                    y->color = BLACK;
                    z->parent->parent->color = RED;
                    z = z->parent->parent;
                } else {
                    if (z == z->parent->right) {
                        // 情况2
                        z = z->parent;
                        leftRotate(z);
                    }
                    // 情况3
                    z->parent->color = BLACK;
                    z->parent->parent->color = RED;
                    rightRotate(z->parent->parent);
                }
            } else {
                // 父节点是右子节点，情况对称
                RBTreeNode<Key, Value>* y = z->parent->parent->left; // 叔叔节点
                if (y != nullptr && y->color == RED) {
                    // 情况1
                    z->parent->color = BLACK;
                    y->color = BLACK;
                    z->parent->parent->color = RED;
                    z = z->parent->parent;
                } else {
                    if (z == z->parent->left) {
                        // 情况2
                        z = z->parent;
                        rightRotate(z);
                    }
                    // 情况3
                    z->parent->color = BLACK;
                    z->parent->parent->color = RED;
                    leftRotate(z->parent->parent);
                }
            }
        }
        root->color = BLACK;
    }

    void inorderHelper(RBTreeNode<Key, Value>* node) const {
        if (node == nullptr) return;
        inorderHelper(node->left);
        std::cout << node->key << " ";
        inorderHelper(node->right);
    }

public:
    RedBlackTree() : root(nullptr) {}

    RBTreeNode<Key, Value>* getRoot() const { return root; }

    void insert(const Key& key, const Value& value) {
        RBTreeNode<Key, Value>* z = new RBTreeNode<Key, Value>(key, value);
        RBTreeNode<Key, Value>* y = nullptr;
        RBTreeNode<Key, Value>* x = root;

        while (x != nullptr) {
            y = x;
            if (z->key < x->key)
                x = x->left;
            else
                x = x->right;
        }

        z->parent = y;
        if (y == nullptr)
            root = z;
        else if (z->key < y->key)
            y->left = z;
        else
            y->right = z;

        // 插入后修正红黑树性质
        insertFixUp(z);
    }

    void inorderTraversal() const {
        inorderHelper(root);
        std::cout << std::endl;
    }

    // 为简化示例，删除操作未实现
    // 完整实现需要包含 deleteFixUp 等步骤
};
```



### 简要说明



上述红黑树类包含以下主要功能：



1. **插入操作 (`insert`)**：
   - 插入新的键值对，并调用 `insertFixUp` 进行修正，以保持红黑树的性质。
2. **旋转操作 (`leftRotate` 和 `rightRotate`)**：
   - 通过旋转操作重新调整树的结构，确保树的平衡。
3. **修正插入后的红黑树性质 (`insertFixUp`)**：
   - 根据红黑树的五大性质，通过重新着色和旋转来修正可能的违规情况。
4. **中序遍历 (`inorderTraversal`)**：
   - 以中序遍历的方式输出树中的键，结果应为升序。



> **注意**：为了简化示例，删除操作 (`delete`) 及其修正 (`deleteFixUp`) 未在此实现。如果需要完整的删除功能，请参考之前的详细解释或使用标准库中的实现。

## 6. 红黑树与其他平衡树的比较



红黑树并非唯一的自平衡二叉搜索树，其他常见的平衡树包括 **AVL 树**（Adelson-Velsky和Landis树）和 **Splay 树**。以下是红黑树与 AVL 树的比较：



### 红黑树 vs AVL 树



| 特性              | 红黑树 (Red-Black Tree)                                | AVL 树 (AVL Tree)                                            |
| ----------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **平衡性**        | 相对不严格，每个路径上的黑色节点相同。                 | 更严格，任意节点的左右子树高度差不超过1。                    |
| **插入/删除效率** | 较快，插入和删除操作较少的旋转，适用于频繁修改的场景。 | 较慢，插入和删除可能需要多次旋转，适用于查找操作多于修改的场景。 |
| **查找效率**      | O(log n)                                               | O(log n)，常数因子更小，查找速度略快。                       |
| **实现复杂度**    | 相对简单，旋转操作较少。                               | 实现较复杂，需严格维护高度平衡。                             |
| **应用场景**      | 操作频繁、需要快速插入和删除的场景。                   | 查找操作频繁、插入和删除相对较少的场景。                     |



### 选择依据



- **红黑树**更适用于需要频繁插入和删除操作，并且查找操作相对较多的场景，因为其插入和删除操作的调整成本较低。
- **AVL 树**适用于查找操作极为频繁，而修改操作相对较少的场景，因为其高度更严格，查找效率更高。





## 7. 红黑树的应用场景



由于红黑树高效的查找、插入和删除性能，它在计算机科学中的多个领域都有广泛的应用：



1. **标准库中的关联容器**：
   - C++ 标准库中的 `std::map` 和 `std::set` 通常基于红黑树实现。
   - Java 的 `TreeMap` 和 `TreeSet` 也是基于红黑树。
2. **操作系统**：
   - Linux 内核中的调度器和虚拟内存管理使用红黑树来管理进程和内存资源。
3. **数据库系统**：
   - 一些数据库索引结构使用红黑树来提高查询效率。
4. **编译器设计**：
   - 语法分析树和符号表管理中可能使用红黑树来高效存储和查找符号。



