---
title: RAII
date: 2025-11-18 22:44:28
categories: 编程语言
tags: C++
---

> Q: 什么是 RAII？RAII 如何保证资源安全释放？
>
> A: RAII（Resource Acquisition Is Initialization，资源获取即初始化）是一种 C++ 编程思想：**把资源（如内存、文件句柄等）的管理交给对象的生命周期**。当对象生命周期结束（无论是正常退出还是异常抛出）时，资源都能自动释放，避免内存泄漏。
>
> 核心思想是：
>
> - 在对象构造函数中获取资源；
> - 在析构函数中释放资源。
>
> 典型例子包括：
>
> - 智能指针自动释放内存；
> - lock_guard 自动释放锁。
>

------

## 🧩 什么是 RAII？

RAII 是 C++ 的核心设计理念之一，全称：

> **Resource Acquisition Is Initialization** —— 资源获取即初始化。

它的作用是：

- 把资源的申请与对象的构造绑定；
- 把资源的释放与对象的析构绑定。

因此，只要对象的生命周期结束（无论以何种方式），资源一定被安全释放。

------

## 🧱 为什么需要 RAII？

在早期 C 语言或不良 C++ 代码中，常见这种问题：

```cpp
FILE* fp = fopen("data.txt", "r");
if (!fp) {
    return;
}
fread(buffer, 1, 100, fp);
// ...
if (error) {
    return;  // 忘记 fclose() → 文件句柄泄漏
}
fclose(fp);
```

如果函数提前 `return`，资源没被释放。

RAII 的解决方案：

```cpp
#include <cstdio>

class FileWrapper {
    FILE* fp;
public:
    FileWrapper(const char* name, const char* mode) {
        fp = fopen(name, mode);
        if (!fp) {
            throw std::runtime_error("open failed");
        }
    }
    ~FileWrapper() { 
        if (fp) {
            fclose(fp); 
        }
    }

    FILE* get() const { 
        return fp; 
    }
};

void readFile() {
    FileWrapper file("data.txt", "r");  // 构造时打开
    char buf[100];
    fread(buf, 1, 100, file.get());     // 使用中可能抛异常
}   // 离开作用域自动 fclose()
```

无论函数是否异常退出，文件一定被关闭。

------

## ⚙️ RAII 的核心机制

| 阶段     | 发生的事                                                     |
| -------- | ------------------------------------------------------------ |
| 构造函数 | 获取资源（分配内存、打开文件、加锁）                         |
| 析构函数 | 释放资源（`delete`、`close`、`unlock`）                      |
| 异常情况 | C++ 会自动调用栈上对象的析构函数，确保释放（C++ 的「栈展开（stack unwinding）」机制） |

------

## 💡 典型 RAII 实例

### `std::unique_ptr` / `std::shared_ptr`

```cpp
{
    auto ptr = std::make_unique<int>(10);  // 构造时分配
    // ...
} // 离开作用域，自动 delete
```

**构造函数**：申请堆内存
**析构函数**：释放内存

------

### `std::lock_guard` —— 线程锁保护

```cpp
std::mutex m;
void thread_func() {
    std::lock_guard<std::mutex> lock(m);  // 加锁
    // 临界区
} // 自动解锁
```

**构造函数**：`lock()`
**析构函数**：`unlock()`

------

## 🧠 RAII 如何保证资源安全释放？

### 栈展开机制

C++ 在作用域结束时，会自动调用局部对象的析构函数。当程序抛出异常（`throw`）后，运行时系统会**依次销毁当前调用栈上的局部对象（调用它们的析构函数）**，直到找到能处理该异常的 `catch` 块为止。它让 **C++ 在异常发生时仍然能自动释放资源**，这是 **RAII 能成立的底层基础机制**。

```cpp
void foo() {
    std::lock_guard<std::mutex> guard(mtx);
    throw std::runtime_error("oops");  // guard 的析构仍会执行！
}
```

### 栈展开的定义

**Stack Unwinding（栈展开）** 指的是：当函数在执行中抛出异常后，C++ 运行时会沿着调用栈逐层回退，依次：

1. 调用已构造对象的析构函数；
2. 弹出栈帧；
3. 直到找到能处理异常的 `catch` 块；
4. 若没找到，程序最终调用 `std::terminate()` 结束。

------

### 底层执行流程

1. `throw` 语句生成异常对象；
2. 程序运行时在异常点向上查找：
    - 当前函数有没有 `try-catch`；
    - 没有 → 回退一层；
3. 每回退一层：
    - 调用该函数栈帧里所有**已构造的局部对象析构函数**；
    - 清理栈内存；
4. 一旦找到能匹配异常类型的 `catch` 块：
    - 执行对应 `catch`；
    - 结束栈展开过程；
5. 若到主函数都没 catch：
    - 调用 `std::terminate()` → 程序终止。

------

### 与 RAII 的关系

RAII（资源获取即初始化）的关键思想是：资源的释放放在析构函数里。栈展开机制保证析构函数一定会被调用，即使是异常退出。

**示例：**

```cpp
struct File {
    FILE* fp;
    File(const char* name) { fp = fopen(name, "r"); }
    ~File() { if (fp) fclose(fp); }
};

void readFile() {
    File file("data.txt");     // 构造时打开
    throw std::runtime_error("oops");  // 异常
}   // 栈展开：自动调用 file.~File() → fclose()
```

> ✅ 即使抛异常，`fclose()` 仍然执行。这正是 C++ 能实现「异常安全」的根基。

------

### 对象构造失败时的栈展开

如果构造函数内部抛出异常：

- 只会销毁**已成功构造的成员**；
- 未完成构造的部分不会析构；
- 外层对象也不会进入有效状态。

```cpp
struct A { A() { throw 1; } };
struct B { A a; ~B() { std::cout << "~B\n"; } };

int main() {
    try {
        B b;  // 构造 A 时抛出异常
    } catch (...) {}
}
```

输出：

```
（什么都不打印）
```

> 因为 `B` 的构造未完成，不会调用 `~B()`；
> `A` 也没构造成功，不需要析构。

------

### 编译器实现原理（简略）

编译器在生成代码时，会为每个可能抛异常的函数维护**异常处理表**，包括：

- 哪些局部变量有析构；
- 对应析构函数地址；
- 哪些范围匹配哪些 catch；

当 `throw` 时，运行时查表并依次调用析构；如果没有异常那么和正常运行一样高效，这就是「zero-cost exception model」（不抛异常时零开销）。

------

### 注意事项

1. **析构函数不要再抛异常**

    - 栈展开中若析构再抛异常 → `std::terminate()`；

    ```cpp
    ~A() noexcept { /* safe */ }  // 推荐
    ```

2. **手动禁用栈展开**

    - 某些高性能模块（如音视频实时线程）禁用异常（`-fno-exceptions`），手动管理错误返回值。

3. **多线程环境**

    - 异常仅限当前线程传播，不跨线程。

------

## 🚀 RAII 的优势总结
| 优点           | 说明                                       |
| -------------- | ------------------------------------------ |
| ✅ 自动释放资源 | 避免内存泄漏                               |
| ✅ 异常安全     | 抛异常也能正确释放                         |
| ✅ 可组合       | 可与 STL 容器、智能指针等结合              |
| ✅ 可移植       | 适用于任何需要 acquire/release 的资源      |
| ⚙️ 可扩展       | 可自定义类来管理特定资源（如文件、socket） |

## 📘 常见 RAII 模式
| RAII 类            | 管理资源         | 析构时动作     |
| ------------------ | ---------------- | -------------- |
| `std::unique_ptr`  | 动态内存         | `delete`       |
| `std::shared_ptr`  | 动态内存（共享） | `delete`       |
| `std::lock_guard`  | `std::mutex`     | `unlock()`     |
| `std::scoped_lock` | 多锁管理         | `unlock_all()` |
| `std::fstream`     | 文件             | `close()`      |
