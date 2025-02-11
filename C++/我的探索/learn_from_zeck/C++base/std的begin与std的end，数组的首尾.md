	# 先例

**使用数组初始化vector对象**

介绍过不允许使用一个数组为另一个内置类型的数组赋初值，也不允许使用vector对象初始化数组。

相反的，允许使用数组来初始化vector对象。要实现这一目的，**只需指明要拷贝区域的首元素地址和尾后地址就可以了：**

``` cpp
int int_arr[] = {0,1,2,3,4,5};
std::vector<int> ivec(std::begin(int_arr), std::end(int_arr));
for(auto e : ivec){
    std::cout << e << " ";
}
```

而实际上，对于我们之前讲到的`std::vector::begin`和`std::vector::end `，这里的`std::begin`和`std::end`会直接调用它们。



在C++中，`std::begin` 和 `std::end` 是两个实用函数模板，它们提供了一种通用的方式来获取容器或数组的开始和结束迭代器。以下是它们的详细介绍和常用用法：

------

### **1. 基本功能**

- `std::begin(container)`

  :

  - 返回容器或数组的起始迭代器，指向第一个元素。

- `std::end(container)`

  :

  - 返回容器或数组的结束迭代器，指向最后一个元素的**后一个位置**。

它们的主要优点是统一接口，无论是标准容器（如`std::vector`）还是普通数组，都可以使用这两个函数获取迭代器。

------

### **2. 常见用法**

#### **（1）在范围 for 循环中使用**

在 C++11 引入范围 `for` 循环后，`std::begin` 和 `std::end` 主要用于实现迭代范围。

```cpp
#include <iostream>
#include <vector>
#include <array>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::array<int, 3> arr = {10, 20, 30};

    // 使用 std::begin 和 std::end 显式循环
    for (auto it = std::begin(vec); it != std::end(vec); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 普通数组
    int raw_arr[] = {100, 200, 300};
    for (auto it = std::begin(raw_arr); it != std::end(raw_arr); ++it) {
        std::cout << *it << " ";
    }
    return 0;
}
```

#### **（2）在算法中配合使用**

`std::begin` 和 `std::end` 常与 STL 算法一起使用，比如 `std::sort`、`std::find` 等。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> vec = {5, 3, 4, 1, 2};

    // 使用 std::sort 排序
    std::sort(std::begin(vec), std::end(vec));

    // 输出结果
    for (const auto& val : vec) {
        std::cout << val << " ";
    }

    return 0;
}
```

#### **（3）适用于普通数组**

普通数组不像标准容器那样自带 `begin()` 和 `end()` 成员函数，因此 `std::begin` 和 `std::end` 对数组特别有用。

```cpp
#include <iostream>
#include <algorithm>

int main() {
    int arr[] = {4, 1, 3, 5, 2};

    // 对普通数组排序
    std::sort(std::begin(arr), std::end(arr));

    for (int x : arr) {
        std::cout << x << " ";
    }

    return 0;
}
```

------

### **3. 它们和其他方式的对比**

#### **与成员函数 `begin()` 和 `end()` 的关系**

对于标准容器（如 `std::vector`, `std::list` 等），`std::begin` 和 `std::end` 实际上调用了容器的成员函数 `begin()` 和 `end()`。

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3};

    // 等价操作
    auto it1 = vec.begin();     // 成员函数
    auto it2 = std::begin(vec); // std::begin 调用成员函数

    std::cout << *it1 << " " << *it2 << std::endl;

    return 0;
}
```

#### **支持普通数组**

普通数组没有成员函数 `begin()` 和 `end()`，但可以直接使用 `std::begin` 和 `std::end`。

```cpp
int arr[] = {1, 2, 3, 4};
auto first = std::begin(arr);  // 指向 arr[0]
auto last = std::end(arr);     // 指向 arr[4] (越界位置)
```

#### **与传统指针的对比**

普通数组的起始和结束位置也可以用指针表示：

```cpp
int arr[] = {1, 2, 3, 4};
int* start = arr;
int* finish = arr + 4;  // 等同于 std::end(arr)
```

但使用 `std::begin` 和 `std::end` 更直观且安全，特别是在模板代码中。

------

### **4. 注意事项**

1. **适用于 C++11 及更高版本**： 需要在代码中包含头文件 `<iterator>`。

2. **对于动态数组（如通过 `new` 分配的数组）无效**： `std::begin` 和 `std::end` 只适用于普通数组和标准容器。

   ```cpp
   int* dyn_arr = new int[5]{1, 2, 3, 4, 5};
   // std::begin(dyn_arr) 无效，因为它不是数组的类型
   ```

3. **统一接口在泛型代码中非常有用**： 无需区分容器和数组，`std::begin` 和 `std::end` 让编写泛型代码更简单。

------

### **5. 总结**

- `std::begin` 和 `std::end` 提供了一种统一获取迭代范围的方式。
- 适用于标准容器和普通数组，但不支持动态数组。
- 它们经常与 STL 算法结合使用，能简化代码书写和提高代码的通用性。