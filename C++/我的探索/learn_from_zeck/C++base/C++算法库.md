# std::partition
`std::partition` 是 C++ 标准库中的一个算法，用于对容器中的元素进行重新排列，使得满足特定条件的元素排在前面，不满足条件的元素排在后面。它定义在 `<algorithm>` 头文件中。

### 函数原型

```cpp
template< class ForwardIt, class UnaryPredicate >
ForwardIt partition( ForwardIt first, ForwardIt last, UnaryPredicate p );
```

- `first` 和 `last` 是迭代器，表示要处理的范围 `[first, last)`。
- `p` 是一个一元谓词（即接受一个参数并返回 `bool` 的函数或函数对象），用于判断元素是否满足条件。

### 返回值

`std::partition` 返回一个迭代器，指向第一个不满足条件的元素。换句话说，所有满足条件的元素都在这个迭代器之前，所有不满足条件的元素都在这个迭代器之后。

### 工作原理

`std::partition` 会重新排列 `[first, last)` 范围内的元素，使得所有满足谓词 `p` 的元素都排在前面，不满足 `p` 的元素都排在后面。它不保证元素的相对顺序保持不变。

### 示例

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

bool is_even(int n) {
    return n % 2 == 0;
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9};

    auto it = std::partition(v.begin(), v.end(), is_even);

    std::cout << "Partitioned vector: ";
    for (int i : v) {
        std::cout << i << " ";
    }
    std::cout << "\n";

    std::cout << "First odd element: " << *it << "\n";

    return 0;
}
```

### 输出

```
Partitioned vector: 8 2 6 4 5 3 7 1 9 
First odd element: 5
```

在这个例子中，`std::partition` 将偶数排在前面，奇数排在后面。返回的迭代器 `it` 指向第一个奇数 `5`。

### 注意事项

1. **不稳定性**：`std::partition` 不保证元素的相对顺序保持不变。如果需要保持相对顺序，可以使用 `std::stable_partition`。
2. **复杂度**：`std::partition` 的时间复杂度是线性的，即 O(n)，其中 n 是 `[first, last)` 范围内的元素数量。

### 与 `std::stable_partition` 的区别

- `std::partition` 不保证元素的相对顺序。
- `std::stable_partition` 保证元素的相对顺序不变，但可能会更慢一些。

### 总结

`std::partition` 是一个非常有用的算法，适用于需要根据某个条件将元素分成两部分的场景。如果你不关心元素的相对顺序，`std::partition` 是一个高效的选择。