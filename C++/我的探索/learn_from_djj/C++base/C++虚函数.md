在C++中，虚函数（virtual function）用于实现多态性。多态性允许你通过基类的指针或引用来调用派生类的函数。如果没有使用虚函数，编译器会根据指针或引用的类型来决定调用哪个函数，而不是根据实际对象的类型。

### 例子说明

假设我们有一个基类 `Base` 和一个派生类 `Derived`，它们都有一个同名的方法 `show()`。

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    void show() {
        cout << "Base class show function" << endl;
    }
};

class Derived : public Base {
public:
    void show() {
        cout << "Derived class show function" << endl;
    }
};

int main() {
    Base* basePtr;
    Derived derivedObj;
    basePtr = &derivedObj;

    // 调用基类的show函数，因为指针类型是Base*
    basePtr->show();

    return 0;
}
```

在这个例子中，`basePtr` 是一个指向 `Base` 类的指针，但它实际上指向了一个 `Derived` 类的对象。当我们调用 `basePtr->show()` 时，输出是：

```
Base class show function
```

这是因为 `show()` 方法在基类中不是虚函数，所以编译器根据指针的类型（`Base*`）来决定调用哪个版本的 `show()` 方法，而不是根据实际对象的类型（`Derived`）。

### 使用虚函数

为了让编译器根据实际对象的类型来决定调用哪个版本的 `show()` 方法，我们需要将 `show()` 方法声明为虚函数。

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    virtual void show() {
        cout << "Base class show function" << endl;
    }
};

class Derived : public Base {
public:
    void show() override {
        cout << "Derived class show function" << endl;
    }
};

int main() {
    Base* basePtr;
    Derived derivedObj;
    basePtr = &derivedObj;

    // 调用派生类的show函数，因为show是虚函数
    basePtr->show();

    return 0;
}
```

在这个修改后的例子中，`show()` 方法在基类中被声明为虚函数（`virtual`）。当我们再次调用 `basePtr->show()` 时，输出是：

```
Derived class show function
```

这是因为 `show()` 是虚函数，编译器会根据实际对象的类型（`Derived`）来决定调用哪个版本的 `show()` 方法，而不是根据指针的类型（`Base*`）。

### 总结

- **非虚函数**：调用哪个版本的函数取决于指针或引用的类型。
- **虚函数**：调用哪个版本的函数取决于实际对象的类型。

通过使用虚函数，你可以实现多态性，使得基类的指针或引用能够调用派生类的函数。