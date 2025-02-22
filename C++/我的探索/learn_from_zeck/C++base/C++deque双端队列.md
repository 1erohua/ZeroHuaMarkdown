## `deque`：双端队列



### 用法



`deque`（双端队列）是一种支持在两端高效插入和删除元素的序列容器。与`vector`相比，`deque`支持在前端和后端均以常数时间进行插入和删除操作。



### 内部实现原理



`deque`通常由一系列固定大小的数组块组成，这些块通过一个中央映射数组进行管理。这种结构使得在两端扩展时不需要重新分配整个容器的内存，从而避免了`vector`在前端插入的高成本。



### 性能特性



- **随机访问**：支持常数时间的随机访问（`O(1)`）。
- **前后插入/删除**：在前端和后端插入和删除元素的操作都是常数时间（`O(1)`）。
- **中间插入/删除**：在中间位置插入或删除元素需要移动元素，时间复杂度为线性时间（`O(n)`）。



### 应用场景



- 需要在容器两端频繁插入和删除元素。
- 需要随机访问元素。
- 不需要频繁在中间位置插入和删除元素。



## 双端队列简介



**双端队列（deque）**是一种序列容器，允许在其两端高效地插入和删除元素。与`vector`不同，`deque`不仅支持在末尾添加或删除元素（如`vector`），还支持在头部进行同样的操作。此外，`deque`提供了随机访问能力，可以像`vector`一样通过索引访问元素。



### 主要特点



- **双端操作**：支持在头部和尾部高效的插入和删除操作。
- **随机访问**：可以像数组和`vector`一样通过索引访问元素。
- **动态大小**：可以根据需要增长和收缩，无需预先定义大小。



------



### 内存分配策略



`deque`内部并不使用一个单一的连续内存块，而是将元素分割成多个**固定大小的块**（也称为**缓冲区**或**页面**），并通过一个**中央映射数组**（通常称为**map**）来管理这些块。具体来说，`deque`的内部结构可以分为以下几个部分：



1. **中央映射数组（Map）**：
   - 一个指针数组，指向各个数据块。
   - `map`本身也是动态分配的，可以根据需要增长或收缩。
   - `map`允许`deque`在两端添加新的数据块，而无需移动现有的数据块。
2. **数据块（Blocks）**：
   - 每个数据块是一个固定大小的连续内存区域，用于存储元素。
   - 数据块的大小通常与编译器和平台相关，但在大多数实现中，数据块的大小在运行时是固定的（如512字节或更多，具体取决于元素类型的大小）。
3. **起始和结束指针**：
   - `deque`维护指向中央映射数组中**第一个有效数据块的指针**以及**第一个无效数据块的指针。**
   - 这些指针帮助`deque`快速地在两端添加或删除数据块。



![https://cdn.llfc.club/912f900f6a609de906df07ee849a57f.png](https://cdn.llfc.club/912f900f6a609de906df07ee849a57f.png)



### 双端队列的操作实现



#### 插入操作



**在末尾插入 (`push_back`)**



1. **检查当前末端数据块的剩余空间**：
   - 如果有空间，直接在当前末端数据块中插入新元素。
   - 如果没有空间，分配一个新的数据块，并将其指针添加到`map`中，然后在新块中插入元素。
2. **更新末尾指针**：
   - 如果分配了新块，末尾指针指向该块的第一个元素。
   - 否则，末尾指针移动到当前末端数据块的下一个位置。



**在前端插入 (`push_front`)**



1. **检查当前前端数据块的剩余空间**：
   - 如果有空间，直接在当前前端数据块中插入新元素。
   - 如果没有空间，分配一个新的数据块，并将其指针添加到`map`的前面，然后在新块中插入元素。
2. **更新前端指针**：
   - **如果分配了新块，前端指针指向该块的最后一个元素。**
   - **否则，前端指针移动到当前前端数据块的前一个位置。**



#### 删除操作



**从末尾删除 (`pop_back`)**



1. 检查末端数据块是否有元素

   ：

   - 如果有，移除最后一个元素，并更新末尾指针。
   - 如果数据块变为空，释放该数据块并从`map`中移除其指针，然后更新末尾指针指向前一个块。



**从前端删除 (`pop_front`)**



1. 检查前端数据块是否有元素

   ：

   - 如果有，移除第一个元素，并更新前端指针。
   - 如果数据块变为空，释放该数据块并从`map`中移除其指针，然后更新前端指针指向下一个块。



#### 访问操作



**随机访问**



`deque`支持通过索引进行随机访问，其内部机制如下：



1. **计算元素的位置**：
   - 根据给定的索引，确定对应的**数据块**和**数据块内的偏移量**。
   - 使用`map`数组定位到具体的块，然后通过偏移量定位到块内的元素。
2. **访问元素**：
   - 一旦定位到具体的位置，即可像数组一样访问元素。



**迭代器访问**



`deque`提供双向迭代器，支持使用标准的C++迭代器操作（如`++`、`--`等）进行遍历。



------



### 双端队列的性能特性



理解`deque`的内部实现有助于理解其性能特性。以下是`deque`的主要操作及其时间复杂度：



| 操作         | 时间复杂度        | 说明                     |
| ---------- | ------------ | ---------------------- |
| 随机访问（通过索引） | 常数时间（O(1)）   | 通过计算块和偏移量直接访问元素        |
| 插入/删除前端    | 常数时间（O(1)）   | 仅涉及前端指针和可能的数据块分配       |
| 插入/删除末端    | 常数时间（O(1)）   | 仅涉及末端指针和可能的数据块分配       |
| 中间插入/删除    | 线性时间（O(n)）   | 需要移动数据块内的元素，可能涉及多个块的操作 |
| 查找元素       | 线性时间（O(n)）   | 需要遍历元素进行查找             |
| 插入单个元素     | 平均常数时间（O(1)） | 在前端或末端插入，通常不需移动大量元素    |
| 插入大量元素     | 线性时间（O(n)）   | 需要分配新的数据块并进行元素复制       |



### 优缺点



#### 优点



- **双端操作高效**：在两端插入和删除元素非常快速，不需要移动大量元素。
- **支持随机访问**：可以像`vector`一样通过索引高效访问元素。
- **动态增长**：无需预先定义大小，可以根据需要自动调整。



#### 缺点



- **内存碎片**：由于使用多个数据块，可能导致内存碎片，尤其是在大量插入和删除操作后。
- **较低的局部性**：元素不连续存储，可能导致缓存未命中率较高，影响性能。
- **复杂性较高**：内部实现相对复杂，不如`vector`直接高效。



------



## 双端队列与其他容器的比较



| 特性          | `vector`                         | `deque`                                     | `list`                                        |
| ------------- | -------------------------------- | ------------------------------------------- | --------------------------------------------- |
| 内存结构      | 单一连续内存块                   | 多块连续内存，通过映射数组管理              | 双向链表                                      |
| 随机访问      | 是，常数时间（O(1)）             | 是，常数时间（O(1)）                        | 否，需要线性时间（O(n)）                      |
| 前端插入/删除 | 低效，线性时间（O(n)）           | 高效，常数时间（O(1)）                      | 高效，常数时间（O(1)）                        |
| 末端插入/删除 | 高效，常数时间（O(1)）           | 高效，常数时间（O(1)）                      | 高效，常数时间（O(1)）                        |
| 内存碎片      | 低，由于单一连续内存块           | 较高，由于多块内存管理                      | 较高，由于节点分散在内存中                    |
| 元素隔离      | 高，局部性较好                   | 中等，分块存储提高了部分局部性              | 低，元素分散存储，缓存效率低                  |
| 应用场景      | 需要频繁随机访问、末端操作的场景 | 需要频繁在两端插入/删除且偶尔随机访问的场景 | 需要频繁在中间插入/删除且不需要随机访问的场景 |



### 代码示例



```cpp
#include <iostream>
#include <deque>

int main() {
    // 创建一个空的deque
    std::deque<std::string> dq;

    // 在末尾添加元素
    dq.push_back("End1");
    dq.push_back("End2");

    // 在前端添加元素
    dq.push_front("Front1");
    dq.push_front("Front2");

    // 遍历deque
    std::cout << "deque中的元素: ";
    for(auto it = dq.begin(); it != dq.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 访问首尾元素
    std::cout << "首元素: " << dq.front() << std::endl;
    std::cout << "尾元素: " << dq.back() << std::endl;

    // 删除首元素
    dq.pop_front();

    // 删除尾元素
    dq.pop_back();

    // 打印删除后的deque
    std::cout << "删除首尾元素后: ";
    for(auto num : dq) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```



### 输出



```makefile
deque中的元素: Front2 Front1 End1 End2 
首元素: Front2
尾元素: End2
删除首尾元素后: Front1 End1 
```
