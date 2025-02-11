
```c++
#include <iostream>
#include <vector>
#include <string>

// 前向声明
class Test;

// 定一个类，它的函数会有一个被作为友元函数
class GetClassName{
public:
    void GetName(Test& test);
};

// 被放入友元函数的类
class Test{
private:
    int age_;
    std::string name_;

public:
    friend void GetClassName::GetName(Test& test);
    Test(int age, std::string name):age_(age),name_(name){};
};

// 在类外定义，即在Test类后面定义
void GetClassName::GetName(Test& test){
    std::cout << test.name_ << std::endl;
}

int main(){
    GetClassName a;
    Test A(11,"zerohu");
    a.GetName(A);

}
```
上面是我的写法


下面是AI的对于友元函数的教学
在C++中，**友元函数（Friend Function）** 是一种特殊的函数，它能够访问类的私有（`private`）和保护（`protected`）成员，即使它不是该类的成员函数。友元函数的定义和声明通常用于某些特定的场景，比如需要外部函数访问类的私有数据时。

### 1. 友元函数的特点
- **非成员函数**：友元函数不是类的成员函数，但它可以访问类的私有和保护成员。
- **声明方式**：友元函数在类内部声明，使用 `friend` 关键字。
- **不受访问控制符限制**：友元函数的访问权限不受 `public`、`private` 或 `protected` 的限制。
- **不属于类**：友元函数不属于类的成员，因此不能通过类的对象调用（例如 `obj.friendFunction()` 是错误的）。
- **单向关系**：如果类 A 声明类 B 的成员函数为友元，那么类 B 的成员函数可以访问类 A 的私有成员，但类 A 不能访问类 B 的私有成员。

### 2. 友元函数的语法
友元函数的声明需要在类内部使用 `friend` 关键字，定义可以在类外部。

```cpp
class MyClass {
private:
    int privateData;

public:
    MyClass(int data) : privateData(data) {}

    // 声明友元函数
    friend void displayPrivateData(const MyClass& obj);
};

// 定义友元函数
void displayPrivateData(const MyClass& obj) {
    // 可以直接访问类的私有成员
    std::cout << "Private data: " << obj.privateData << std::endl;
}

int main() {
    MyClass obj(42);
    displayPrivateData(obj); // 调用友元函数
    return 0;
}
```

在上面的例子中：
- `displayPrivateData` 是 `MyClass` 的友元函数。
- 它可以直接访问 `MyClass` 的私有成员 `privateData`。

### 3. 友元函数的用途
友元函数通常用于以下场景：
- **操作符重载**：当需要重载操作符（如 `<<`、`>>`、`+` 等）时，友元函数可以方便地访问类的私有数据。
- **非成员函数访问私有数据**：某些全局函数或工具函数需要访问类的私有数据时，可以声明为友元函数。
- **增强封装性**：在某些情况下，友元函数可以避免将类的私有成员暴露给外部，同时允许特定的外部函数访问。

### 4. 友元函数的注意事项
- **破坏封装性**：友元函数可以访问类的私有成员，因此过度使用友元函数可能会破坏类的封装性。
- **谨慎使用**：只有在确实需要时才使用友元函数，避免滥用。
- **不能继承**：友元关系不能被继承。如果类 A 是类 B 的友元，类 B 的子类不会自动成为类 A 的友元。
- **不能传递**：友元关系不能传递。如果类 A 是类 B 的友元，类 B 是类 C 的友元，类 A 不会自动成为类 C 的友元。

### 5. 友元函数的例子：操作符重载（重要）
一个常见的例子是重载 `<<` 操作符，以便可以直接输出类的对象。

```cpp
#include <iostream>
using namespace std;

class MyClass {
private:
    int data;

public:
    MyClass(int d) : data(d) {}

    // 声明友元函数
    friend ostream& operator<<(ostream& os, const MyClass& obj);
};

// 定义友元函数
ostream& operator<<(ostream& os, const MyClass& obj) {
    os << "Data: " << obj.data;
    return os;
}

int main() {
    MyClass obj(100);
    cout << obj << endl; // 输出: Data: 100
    return 0;
}
```

在这个例子中：
- `operator<<` 是 `MyClass` 的友元函数。
- 它可以直接访问 `MyClass` 的私有成员 `data`，并将其输出到流中。

### 6. 友元函数与友元类
除了友元函数，C++ 还支持 **友元类（Friend Class）**。友元类的所有成员函数都可以访问另一个类的私有和保护成员。

```cpp
class FriendClass {
public:
    void accessPrivateData(const MyClass& obj) {
        cout << "Accessing private data: " << obj.privateData << endl;
    }
};

class MyClass {
private:
    int privateData;

public:
    MyClass(int data) : privateData(data) {}

    // 声明友元类
    friend class FriendClass;
};

int main() {
    MyClass obj(99);
    FriendClass fc;
    fc.accessPrivateData(obj); // 输出: Accessing private data: 99
    return 0;
}
```

在这个例子中：
- `FriendClass` 是 `MyClass` 的友元类。
- `FriendClass` 的所有成员函数都可以访问 `MyClass` 的私有成员。

### 7. 总结
- 友元函数是一种特殊的函数，可以访问类的私有和保护成员。
- 友元函数通过 `friend` 关键字在类内部声明。
- 友元函数常用于操作符重载或需要访问私有数据的全局函数。
- 友元函数和友元类都需要谨慎使用，以避免破坏封装性。

希望这个详细的解释对你有帮助！如果有其他问题，欢迎继续提问。