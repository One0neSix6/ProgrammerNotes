---
title: extern 关键字
date: 2025-12-03 10:27:16
categories: 编程语言
tags: C++
---

## extern 关键字

### 作用

`extern` 关键字用于**声明变量或函数**，表示这些变量或函数是在其他文件中定义的，编译器会在**链接阶段**去查找它。

### 使用场景

- **多文件项目**：当一个项目有多个源文件时，如果某个变量或函数在一个源文件中定义，而在其他源文件中引用时，可以使用 `extern` 来声明该变量或函数。
- **避免重复定义**：`extern` 让我们可以在多个源文件中共享同一个变量或函数，而不需要在每个源文件中都定义它。

### 示例

假设我们有两个源文件 `file1.cpp` 和 `file2.cpp`，并且需要共享一个全局变量。

file1.cpp

```cpp
#include <iostream>

int global_var = 100;  // 定义变量

void printGlobalVar() {
    std::cout << "Global variable: " << global_var << std::endl;
}
```

file2.cpp

```cpp
#include <iostream>

extern int global_var;  // 声明变量

void changeGlobalVar() {
    global_var = 200;  // 修改变量
}
```

main.cpp

```cpp
#include <iostream>

extern void printGlobalVar();   // 声明函数
extern void changeGlobalVar();  // 声明函数

int main() {
    printGlobalVar();  // 输出 100
    changeGlobalVar();
    printGlobalVar();  // 输出 200
    return 0;
}
```

### 注意事项

- `extern` 只是声明，不会为变量分配内存。
- 在使用 `extern` 声明变量时，需要确保在其他地方有对应的定义，否则链接器会报错。

------

## extern "C" 关键字

### 作用

`extern "C"` 主要用于**告知 C++ 编译器**使用 C 语言的链接方式，而不是 C++ 默认的方式。C++ 和 C 在函数调用约定和符号名称（mangling）上有所不同。C++ 编译器会对函数名进行**名字修饰**（name mangling），以支持函数重载等特性，而 C 语言则不会进行名字修饰。`extern "C"` 关键字的作用是禁止 C++ 编译器对函数进行名字修饰，以便它可以和 C 编译器链接。

### 使用场景

- **C 和 C++ 互操作**：当 C++ 代码需要调用 C 函数，或者 C 函数库需要被 C++ 程序使用时，需要使用 `extern "C"` 来确保函数名称没有被 C++ 编译器修饰，从而确保正确的链接。
- **库函数导出**：当需要将 C++ 函数暴露给 C 程序调用时，也使用 `extern "C"` 以确保 C 程序可以正确识别这些函数。

### 示例

假设我们有一个 C 语言的函数库，并且希望 C++ 调用它。

example.h

```h
void hello_world();
```

example.c

```c
#include "example.h"

#include <stdio.h>

void hello_world() {
    printf("Hello, world from C!\n");
}
```

C++ 调用 C 函数（main.cpp）

```cpp
// 使用 extern "C" 来声明 C 语言函数
extern "C" {
    #include "example.h"
}

int main() {
    hello_world();  // 调用 C 函数
    return 0;
}
```

