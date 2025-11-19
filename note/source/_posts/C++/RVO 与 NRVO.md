---
title: RVO 与 NRVO
date: 2025-11-19 16:30:47
categories: 编程语言
tags: C++
---

## 🧩 概念定义

| 名称     | 全称                              | 含义                                                         |
| -------- | --------------------------------- | ------------------------------------------------------------ |
| **RVO**  | *Return Value Optimization*       | 返回未命名临时对象时，编译器可以省略拷贝/移动构造，直接在目标对象的存储空间上构造。 |
| **NRVO** | *Named Return Value Optimization* | 返回命名的局部变量（即带变量名的对象）时，编译器可以省略拷贝/移动构造。 |

两者本质上都是**复制省略（Copy Elision）** 的形式。

------

## 🧠 C++ 标准的演变

| C++ 版本     | RVO                                                      | NRVO                                         |
| ------------ | -------------------------------------------------------- | -------------------------------------------- |
| **C++98/03** | 可选优化                                                 | 可选优化                                     |
| **C++11/14** | 可选优化（但引入移动语义，如果不省略，则会调用移动构造） | 可选优化                                     |
| **C++17**    | **强制**：当返回临时对象（prvalue）时，必须省略拷贝/移动 | 可选：当返回命名对象时，仍然由编译器自行决定 |

- C++17 强制 **RVO**，但 **NRVO 仍不保证**。
- 使用 `std::move()` 会关闭 NRVO。

------

## 🧪 示例代码

```cpp
#include <iostream>

using namespace std;

struct A {
    A() { cout << "Default ctor\n"; }
    A(const A&) { cout << "Copy ctor\n"; }
    A(A&&) noexcept { cout << "Move ctor\n"; }
    ~A() { cout << "Dtor\n"; }
};

// RVO：返回未命名临时对象
A makeA1() {
    return A();
}

// NRVO：返回命名的局部变量
A makeA2() {
    A a;
    return a;  // 命名对象
}

// 禁止NRVO：使用 std::move
A makeA3() {
    A a;
    return std::move(a);
}

int main() {
    cout << "=== RVO ===\n";
    A a1 = makeA1();

    cout << "=== NRVO ===\n";
    A a2 = makeA2();

    cout << "=== no NRVO (std::move) ===\n";
    A a3 = makeA3();

    return 0;
}
```

------

## 📊 C++17 下的输出结果
> 以 g++ -std=c++17 为例
```
=== RVO ===
Default ctor
Dtor
=== NRVO ===
Default ctor
Dtor
=== no NRVO (std::move) ===
Default ctor
Move ctor
Dtor
Dtor
```

### 解释：

1. **RVO 部分：**
    - 对象直接构造在 `a1` 的存储位置上；
    - 没有调用拷贝或移动构造；
    - 因此只出现一次构造，一次析构。
2. **NRVO 部分：**
    - 编译器通常会进行优化，直接在 `a2` 的存储空间上构造；
    - 如果编译器支持 NRVO（如 GCC/Clang 默认开启），则同样只调用一次构造；
    - 若未启用 NRVO，则会调用移动构造（C++17 前还可能调用拷贝构造）。
3. **禁用 NRVO：**
    - `std::move(a)` 将 `a` 转为右值；
    - 编译器必须调用移动构造；
    - 因此会输出 “Move ctor”；
    - 最后 `a` 在函数退出时被析构。

------

## ⚙️ C++11/14 下的区别

如果你用 `-std=c++14` 编译：

```
=== RVO ===
Default ctor
Move ctor
Dtor
Dtor
```

C++11/14 的行为是：

- RVO 只是**可选**；
- 若编译器未启用，返回临时对象时仍然会调用移动构造；
- 因此多出一次 `Move ctor` 和一个额外析构。

------

## 🧾 总结对比

| 情况      | 表达式                        | C++17 行为   | C++14 行为   |
| --------- | ----------------------------- | ------------ | ------------ |
| RVO       | `return T();`                 | 强制省略拷贝 | 可能移动     |
| NRVO      | `return localVar;`            | 可选优化     | 可选优化     |
| 禁用 NRVO | `return std::move(localVar);` | 强制移动构造 | 强制移动构造 |

------

## 📘 小结与实践建议

✅ **尽量写出易于触发 RVO/NRVO 的代码：**

```cpp
return MyType();   // 强制 RVO
return value;      // 通常触发 NRVO
```

⚠️ **避免滥用 `std::move()`**
在返回局部变量时使用 `std::move()` 会**阻止 NRVO**，从而产生额外的移动构造。

✅ **RVO 是零开销优化**，可以放心依赖。
 NRVO 也非常普遍（GCC/Clang 都支持），但在性能关键路径上最好不要依赖它的存在。
