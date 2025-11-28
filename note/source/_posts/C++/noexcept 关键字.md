---
title: noexcept 关键字
date: 2025-11-28 18:41:34
categories: 编程语言
tags: C++
---

## noexcept 关键字

C++11 开始，`noexcept` 关键字用于声明一个函数不会抛出任何异常。通过使用 `noexcept`，我们可以显式地告诉编译器和程序员，该函数不会抛出异常，从而有助于编译器进行优化、提高程序的性能以及增强代码的可读性和可维护性。

### 基本语法

`noexcept` 关键字可以在函数声明中使用，也可以在函数定义时使用。它通常用于告知编译器该函数是异常安全的，即函数在执行时不会抛出异常。

```cpp
void myFunction() noexcept {
    // 函数体
}
```

在这个例子中，`myFunction` 函数声明了 `noexcept`，表示它不会抛出任何异常。如果在该函数内部发生了异常，程序会终止。

### 作用

#### 提高性能

通过告诉编译器某个函数不会抛出异常，编译器可以针对这些函数进行更多的优化。举例来说，如果一个函数是 `noexcept`，那么编译器可以不为该函数生成与异常处理相关的代码（例如 `try-catch` 块），从而减小代码大小，提高执行效率。

#### 向其他代码传递保证

在函数接口中声明 `noexcept`，可以向其他开发人员或调用者传递该函数不会抛出异常的保证。这样，在使用该函数时，调用者可以放心地知道该函数不会导致异常传播。

### noexcept 的两种形式

`noexcept` 关键字可以分为两种使用方式：**无条件 `noexcept`** 和 **条件性 `noexcept`**。

#### 无条件 `noexcept`

当函数完全没有可能抛出异常时，可以声明为 `noexcept`。这是最常见的用法。

```cpp
void safeFunction() noexcept {
    // 确保这里没有任何可能抛出的异常
}
```

如果函数内部执行了会抛出异常的操作，例如内存分配失败等，那么编译器会报错。

#### 条件性 noexcept

可以通过条件语句在函数声明中使用 `noexcept`，根据函数内部是否会抛出异常来决定是否使用 `noexcept`。这种方式常用于需要根据模板参数、类型等决定是否使用 `noexcept` 的情况。

```cpp
template <typename T>
void conditionalFunction(T& obj) noexcept(noexcept(obj.someMethod())) {
    obj.someMethod();
}
```

在这个例子中，`noexcept` 后面的表达式 `noexcept(obj.someMethod())` 会根据 `obj.someMethod()` 是否标记为 `noexcept` 来决定 `conditionalFunction` 是否是 `noexcept`。如果 `obj.someMethod()` 是 `noexcept`，则 `conditionalFunction` 也会被标记为 `noexcept`。

### 标准库实现

C++ 标准库中的许多函数也使用了 `noexcept`。例如，`std::move` 和 `std::swap` 都被标记为 `noexcept`，因为它们的实现保证不抛出异常。

```cpp
template <typename T>
void swap(T& a, T& b) noexcept(noexcept(a.swap(b))) {
    a.swap(b);
}
```

在 `std::swap` 中，`noexcept` 的使用是根据 `T` 类型的 `swap` 函数是否标记为 `noexcept` 来决定的。

`noexcept` 关键字也在容器操作中起到重要作用。例如，标准库中的某些容器（如 `std::vector`）的成员函数是 `noexcept` 的，以便提供更高效的异常处理和性能优化。

```cpp
std::vector<int> vec;
vec.push_back(10);
```

如果 `std::vector::push_back` 或其他类似操作没有抛出异常，编译器会利用 `noexcept` 标记来做更好的优化。

### 异常传播机制

`noexcept` 关键字对异常的传播有很大的影响。如果一个函数标记为 `noexcept`，那么如果该函数内部抛出了异常，程序将终止。因为 `noexcept` 函数保证不抛出异常，所以如果它抛出了异常，编译器会认为程序发生了严重错误并直接调用 `std::terminate`。

```cpp
void myFunction() noexcept {
    throw std::runtime_error("An error occurred");
}
```

编译器会在这种情况下发出错误，因为 `myFunction` 声明为 `noexcept`，不允许抛出异常。

### 总结

- **`noexcept`** 是 C++11 引入的关键字，用于声明函数不会抛出任何异常。
- **性能优化**：编译器可以利用 `noexcept` 优化代码，避免生成与异常处理相关的代码。
- **明确的接口保证**：通过标记函数为 `noexcept`，开发者可以明确表达函数不会抛出异常的保证。
- **异常传播**：如果标记为 `noexcept` 的函数内部抛出了异常，程序会终止（调用 `std::terminate`）。
