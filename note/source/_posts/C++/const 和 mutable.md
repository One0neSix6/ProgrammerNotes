---
title: const 和 mutable
date: 2025-11-28 18:05:09
categories: 编程语言
tags: C++
---

# const

## const 关键字

在 C++ 中，`const` 关键字用于声明常量，指示某个变量的值在程序运行期间不可更改。`const` 可以修饰基本类型、指针、引用、成员函数等，并且它的使用有许多细微的区别和规则。

### 基本类型

当 `const` 用于基本类型时，它声明一个常量变量。

```cpp
const int x = 10;  // 定义一个常量整型变量 x，值为 10
x = 20;  // 错误：无法修改常量
```

### 常量指针与指针常量

`const` 关键字在指针类型中有两种常见的用法：**常量指针**和**指针常量**。

1. **常量指针**：指向的值不可修改，但指针本身可以修改。

    ```cpp
    int x = 10;
    const int* ptr = &x;  // 常量指针，指向一个常量
    *ptr = 20;  // 错误：不能修改常量的值
    ptr = nullptr;  // 可以修改指针的值
    ```

2. **指针常量**：指针本身不可修改，但可以修改指向的值。

    ```cpp
    int x = 10;
    int* const ptr = &x;  // 指针常量
    *ptr = 20;  // 可以修改指向的值
    ptr = nullptr;  // 错误：不能修改指针本身
    ```

3. **常量指针常量**：指针和指向的值都不可修改。

    ```cpp
    int x = 10;
    const int* const ptr = &x;  // 常量指针常量
    *ptr = 20;  // 错误：不能修改常量的值
    ptr = nullptr;  // 错误：不能修改指针本身
    ```

### 常量成员函数

`const` 还可以用于成员函数，表示该函数不会修改类的成员变量。常量成员函数不能修改类的任何成员变量（除非它们是 `mutable`）。常量成员函数在声明时使用 `const` 修饰在函数后面：

```cpp
class MyClass {
public:
    void myMethod() const {  // 常量成员函数
        // x = 10;  // 错误：无法修改成员变量
    }

private:
    int x;
};
```

### 常量引用

常量引用常用于传递对象的引用，而不允许在函数中修改对象的状态。使用 `const` 修饰引用，避免了不必要的拷贝，并且保护了对象不被修改：

```cpp
void printValue(const int& value) {
    // value = 20;  // 错误：不能修改常量引用
    std::cout << value << std::endl;
}
```

## constexpr 关键字

`constexpr` 是 C++11 引入的关键字，用于指示一个值或函数在编译时是常量（即编译时求值）。`constexpr` 主要用于常量表达式的计算和编译时优化。它比 `const` 更强大，因为它不仅要求变量值在运行时不可变，而且要求在编译时就已知。

### constexpr 变量

`constexpr` 变量在声明时必须是一个常量表达式，且其值必须在编译时确定。`constexpr` 变量本质上是编译时常量。

```cpp
constexpr int x = 10;  // 编译时常量
constexpr int y = x * 2;  // 编译时常量
```

如果 `constexpr` 变量的值在运行时才能确定，就会导致编译错误：

```cpp
int value = 10;
constexpr int x = value;  // 错误：value 不是编译时常量
```

### constexpr 函数

`constexpr` 函数要求函数在编译时能够计算出结果，因此它必须符合一定的规则：

- `constexpr` 函数中的所有参数必须是常量表达式。
- 函数体内部只能使用编译时常量，并且不能包含动态内存分配、异常等非编译时操作。

```cpp
constexpr int add(int a, int b) {
    return a + b;
}

int main() {
    constexpr int result = add(5, 7);  // 编译时求值
    std::cout << result << std::endl;   // 输出 12
}
```

如果一个函数不是 `constexpr`，但仍然可以在运行时进行常量折叠（即仅用于常量的计算），编译器会根据上下文决定是否将它折叠成常量。

### constexpr 和 const 的区别

- `const` 变量可以在运行时初始化，它不要求在编译时已知值。
- `constexpr` 变量必须在编译时已知值，并且它用于声明编译时常量。

```cpp
const int x = getValue();   // 运行时初始化
constexpr int y = 10;       // 编译时初始化
```

## 标准库中的 const 和 constexpr

C++ 标准库中大量使用了 `const` 和 `constexpr`，尤其是在类型安全、优化和编译时常量表达式方面。

### std::array

`std::array` 是 C++11 中引入的一个容器类，它的大小必须是常量表达式，因此可以与 `constexpr` 一起使用：

```cpp
constexpr size_t size = 10;
std::array<int, size> arr;  // 使用 constexpr 声明大小
```

### std::vector

虽然 `std::vector` 的大小是动态的，但它的元素可以是常量，避免不小心修改：

```cpp
std::vector<int> v = {1, 2, 3};
const int firstElement = v[0];  // const 修饰元素，防止修改
```

### std::string

`std::string` 经常与 `const` 一起使用，以保证字符串不被修改。例如，在函数参数中传递常量字符串：

```cpp
void printString(const std::string& str) {
    std::cout << str << std::endl;
}
```

### std::bitset

`std::bitset` 是一个具有固定大小的位集合，它的大小也是在编译时就已知的，因此通常与 `constexpr` 一起使用：

```cpp
constexpr std::bitset<8> bits("11001010");
std::cout << bits << std::endl;  // 编译时常量
```

### 总结

- **`const`** 用于声明常量、常量指针、常量引用以及常量成员函数，它的关键作用是防止变量的值在程序运行时发生变化。
- **`constexpr`** 用于声明编译时常量，它要求变量或函数在编译时就可以确定值，并且允许在编译时进行计算。
- 在 C++ 标准库中，`const` 和 `constexpr` 被广泛用于优化和类型安全，特别是在 `std::array`、`std::vector`、`std::bitset` 等类型中，确保对象或其内容在编译时是常量。

# mutable

在 C++ 中，`mutable` 关键字是一个独特的工具，用于声明类中的成员变量，使其能够在常量成员函数中被修改。通常，常量成员函数会限制对类成员变量的修改，因为这些函数承诺不改变对象的状态。然而，使用 `mutable` 修饰的成员变量可以突破这一限制，即使在常量成员函数中也能被修改。

## 基本用法

通常情况下，`const` 成员函数承诺不改变类的成员变量，但有时可能需要改变某些数据（如计数器、缓存、状态等）。这时，`mutable` 关键字派上用场。通过将某些成员变量声明为 `mutable`，可以在 `const` 成员函数中修改它们，而不违反 `const` 限制。

```cpp
class MyClass {
public:
    void incrementCount() const {
        ++accessCount;  // 在 const 成员函数中修改 mutable 成员
    }

    int getCount() const {
        return accessCount;  // 获取 accessCount 的值
    }
private:
    mutable int accessCount;  // 允许在 const 成员函数中修改
};

int main() {
    MyClass obj;
    obj.incrementCount();
    std::cout << obj.getCount() << std::endl;  // 输出 1
}
```

`accessCount` 是 `mutable` 的，即使 `incrementCount` 被声明为 `const`，仍然可以修改 `accessCount` 的值。

---


`mutable` 的使用并不限于普通成员变量，指针和引用类型的成员变量也可以使用 `mutable`。这种用法通常用于在 `const` 成员函数中修改指针或引用指向的对象，而不违反 `const` 的限制。

```cpp
class MyClass {
public:
    mutable int* ptr;  // const 成员函数中可以修改这个指针

    void modifyPointer() const {
        *ptr = 42;  // 在 const 成员函数中修改指针指向的值
    }
};
```

## 使用 mutable 的常见场景

`mutable` 通常用于以下几种场景：

- **缓存**：当需要在类中缓存计算结果时，常常会将缓存数据声明为 `mutable`，以便在 `const` 成员函数中更新缓存。
- **延迟计算**：某些情况下，可能需要在 `const` 成员函数中进行延迟计算。使用 `mutable`，可以确保计算结果存储在对象的成员中，而不违反 `const` 限制。
- **计数器**：有时需要在类的常量对象上维护一个计数器，比如跟踪对象被访问的次数，这时可以将计数器声明为 `mutable`，以便在 `const` 成员函数中修改。
- **互斥量**：对于某些成员函数，虽然不会改变成员变量的值，但是需要加锁访问，这里的互斥量就需要使用 `mutable` 修饰，否则会编译报错。

```cpp
class Cache {
public:
    mutable bool isValid;  // 标记缓存是否有效

    void updateCache() const {
        isValid = true;  // 在 const 成员函数中修改 isValid
    }

    void invalidateCache() const {
        isValid = false;  // 在 const 成员函数中修改 isValid
    }
};
```

## 注意事项

尽管 `mutable` 提供了在 `const` 成员函数中修改数据的能力，但它也应该谨慎使用，因为它打破了类的常量性承诺，可能导致程序逻辑上的混淆或潜在的错误。过度使用 `mutable` 可能使得对象的不可变性变得不明确，从而增加调试和维护的难度。

- **不应过度使用**：只在需要在 `const` 成员函数中修改成员变量时使用 `mutable`。避免用它来处理不必要的可变状态，应该尽量保持类的状态在对象生命周期中的一致性。
- **适当文档化**：使用 `mutable` 的类成员变量应清晰地标注其意图，确保代码的可读性和维护性，尤其是在大型项目中。
