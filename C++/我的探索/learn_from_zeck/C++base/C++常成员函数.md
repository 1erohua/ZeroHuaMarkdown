
#### 1. 常成员函数的概念
常成员函数是指在成员函数声明和定义后加上 `const` 关键字的函数。它表示该函数不会修改对象的状态（即不会修改对象的成员变量）。

**语法：**
```cpp
返回类型 函数名(参数列表) const;
```

**示例：**
```cpp
class MyClass {
public:
    void display() const {
        std::cout << "This is a const member function." << std::endl;
    }
};
```

**说明：**
- `display` 是一个常成员函数，它不会修改 `MyClass` 对象的任何成员变量。

#### 2. 常成员函数的特点
- **不能修改成员变量**：常成员函数内部不能修改类的非静态成员变量（除非成员变量被声明为 `mutable`）。
- **只能调用其他常成员函数**：常成员函数内部**只能调用其他常成员函数，不能调用非常成员函数。**（因为其他常成员函数有可能会改变内部成员）
- **可以被常对象调用**：常对象（即被声明为 `const` 的对象）只能调用常成员函数。

**示例：**
```cpp
class MyClass {
public:
    void modify() {
        value = 10;  // 修改成员变量
    }

    void display() const {
        std::cout << "Value: " << value << std::endl;
        // modify();  // 错误：常成员函数不能调用非常成员函数
    }

private:
    int value;
};

int main() {
    const MyClass obj;
    // obj.modify();  // 错误：常对象不能调用非常成员函数
    obj.display();    // 正确：常对象可以调用常成员函数
    return 0;
}
```

**说明：**
- `modify` 是非常成员函数，可以修改成员变量 `value`。
- `display` 是常成员函数，不能修改成员变量 `value`，也不能调用非常成员函数 `modify`。
- `obj` 是常对象，只能调用常成员函数 `display`。

#### 3. `mutable` 关键字
`mutable` 关键字用于修饰类的成员变量，表示即使在常成员函数中也可以修改该成员变量。

**示例：**
```cpp
class MyClass {
public:
    void display() const {
        counter++;  // 修改 mutable 成员变量
        std::cout << "Counter: " << counter << std::endl;
    }

private:
    mutable int counter = 0;  // mutable 成员变量
};

int main() {
    const MyClass obj;
    obj.display();  // 正确：常成员函数可以修改 mutable 成员变量
    return 0;
}
```

**说明：**
- `counter` 被声明为 `mutable`，即使在常成员函数 `display` 中也可以修改它的值。

#### 4. 常成员函数的重载
常成员函数和非常成员函数可以构成重载，编译器根据调用对象的常量性选择合适的函数。

**示例：**
```cpp
class MyClass {
public:
    void display() {
        std::cout << "Non-const member function." << std::endl;
    }

    void display() const {
        std::cout << "Const member function." << std::endl;
    }
};

int main() {
    MyClass obj1;
    const MyClass obj2;

    obj1.display();  // 调用非常成员函数
    obj2.display();  // 调用常成员函数
    return 0;
}
```

**说明：**
- `obj1` 是非常对象，调用非常成员函数 `display`。
- `obj2` 是常对象，调用常成员函数 `display`。

#### 5. 总结
- **常成员函数**：不会修改对象状态的成员函数，声明时在函数后加 `const`。
- **特点**：不能修改成员变量（除非 `mutable`），只能调用其他常成员函数，常对象只能调用常成员函数。
- **`mutable`**：允许在常成员函数中修改的成员变量。
- **重载**：常成员函数和非常成员函数可以构成重载。

常成员函数是 C++ 中保证对象状态不被修改的重要机制，尤其在处理常对象时非常有用。