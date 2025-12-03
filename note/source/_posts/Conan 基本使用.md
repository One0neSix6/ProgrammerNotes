---
title: Conan 基本使用
date: 2025-12-03 11:54:31
categories: 其他
tags: 工具
---

## Conan 2.x 学习笔记

### Conan 是什么

> **Conan** 是一个面向 **C/C++** 的跨平台包管理器。
> 它能自动完成：
>
> - 下载依赖库（如 glfw、boost、fmt、openssl 等）
> - 解决依赖关系和版本冲突
> - 管理编译配置（编译器、标准库、架构）
> - 与构建系统（CMake、MSBuild、Make）自动集成

它的作用类似于：

| 语言    | 包管理工具  |
| ------- | ----------- |
| Python  | pip         |
| Node.js | npm         |
| Rust    | cargo       |
| Java    | Maven       |
| C/C++   | **Conan** ✅ |

------

### Conan 常用命令汇总

#### 查看当前安装的 Conan 版本

```bash
conan --version
```

输出示例：

```text
Conan version 2.20.1
```

------

#### 查看本地配置的远程仓库

```bash
conan remote list
```

示例输出：

```
conancenter: https://center.conan.io [Verify SSL: True]
```

说明：`conancenter` 是 Conan 官方中央仓库（默认已存在）。

------

#### 添加 / 删除 远程仓库

```bash
conan remote add <名称> <URL>
conan remote remove <名称>
```

示例：

```bash
conan remote add conancenter https://center.conan.io
```

------

#### 在远程仓库中查找某个库

```bash
conan search <包名> -r <远程仓库名>
```

示例：

```bash
conan search glfw -r conancenter
```

输出：

```
Existing package recipes:
  glfw/3.3.8
  glfw/3.4
```

------

####  查看已安装的包

```bash
conan list
```

或指定某个包：

```bash
conan list glfw
```

------

#### 安装依赖

```bash
conan install <路径> [参数]
```

常见参数说明：

| 参数                     | 含义                                                       |
| ------------------------ | ---------------------------------------------------------- |
| `.`                      | 指当前目录（即包含 conanfile.txt 或 conanfile.py 的目录）  |
| `--build=missing`        | 如果找不到预编译包，则自动从源码构建                       |
| `--output-folder=<目录>` | 指定生成的工具链配置、依赖信息的输出目录（通常是 `build`） |
| `-s build_type=Release`  | 指定构建类型（Debug / Release）                            |
| `-pr`                    | 指定使用的 profile 文件（比如编译器、平台配置）            |
| `-r conancenter`         | 指定使用的远程仓库                                         |

示例：

```bash
conan install . --output-folder=cmake-build-debug --build=missing -s build_type=Debug
```

------

#### 清理本地缓存

```bash
conan remove "*" -f
```

------

### Conan 配置文件格式

一个最常见的 Conan 配置文件结构如下：

```ini
[requires]
glfw/3.3.8

[generators]
CMakeDeps
CMakeToolchain

[layout]
cmake_layout

[options]
glfw:shared=True
```

各部分含义：

| 部分           | 说明                                   | 示例                                                         |
| -------------- | -------------------------------------- | ------------------------------------------------------------ |
| `[requires]`   | 声明项目依赖的第三方库（包名/版本）    | `glfw/3.3.8`                                                 |
| `[generators]` | 指定 Conan 如何生成给 CMake 使用的文件 | `CMakeDeps`（生成包信息）`CMakeToolchain`（生成工具链文件）  |
| `[layout]`     | 指定项目目录结构布局方式               | `cmake_layout` 自动生成标准目录结构（`build/Release/generators`） |
| `[options]`    | 自定义依赖的编译选项                   | 比如：`glfw:shared=True` 表示构建为动态库                    |

------

### CMake 配置

假设你的项目结构：

```
OpenGLStudy/
├── conanfile.txt
├── CMakeLists.txt
└── main.cpp
```

#### 运行 Conan 安装依赖

```bash
conan install . --output-folder=cmake-build-debug --build=missing -s build_type=Debug
conan install . --output-folder=cmake-build-release --build=missing -s build_type=Release
```

生成目录：

```
cmake-build-debug/build/
 ├── conan_toolchain.cmake
 ├── glfw3-config.cmake
 └── ...
```

------

#### CMakeLists.txt 示例

```cmake
cmake_minimum_required(VERSION 3.5)
cmake_policy(SET CMP0091 NEW)

if (MSVC)
    add_compile_options(/utf-8)
endif()

project(OpenGLStudy)

set(CMAKE_CXX_STANDARD 20)

# 生成可执行文件
add_executable(OpenGLStudy main.cpp)

# 查找由 Conan 提供的 GLFW 包（CMakeDeps 生成的）
find_package(glfw3 REQUIRED)

# 链接 GLFW
target_link_libraries(OpenGLStudy PRIVATE glfw)
```

------

### CMake 与 Conan 的工作机制

| 组件                        | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **CMakeDeps**               | 为每个依赖生成 `xxx-config.cmake` 文件，供 `find_package()` 使用 |
| **CMakeToolchain**          | 自动设置编译器路径、构建类型、依赖路径等                     |
| **cmake_layout**            | 统一项目目录结构（如 `build/Release/generators/`）           |
| **find_package()**          | 从 Conan 生成的配置文件中找到库                              |
| **target_link_libraries()** | 将目标程序与 Conan 提供的库目标连接起来                      |

### 完整示例

#### conanfile.txt

```ini
[requires]
glfw/3.3.8

[generators]
CMakeDeps
CMakeToolchain

[layout]
cmake_layout
```

#### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.5)
cmake_policy(SET CMP0091 NEW)
project(OpenGLStudy)

if (MSVC)
    add_compile_options(/utf-8)
endif()

set(CMAKE_CXX_STANDARD 20)

find_package(glfw3 REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)

target_link_libraries(${PROJECT_NAME} PRIVATE glfw)
```

#### main.cpp

```cpp
#include <iostream>
#include <GLFW/glfw3.h>

int main() {
    if (!glfwInit()) {
        std::cerr << "GLFW init failed\n";
        return -1;
    }
    std::cout << "GLFW init success!\n";
    glfwTerminate();
    return 0;
}
```
#### 构建项目

CLion 配置——构建、执行、部署——CMake

工具链：VS

生成器：VS 2022

构建目录：cmake-build-debug

```shell
# CMake 选项
-G "Visual Studio 17 2022"
-DCMAKE_TOOLCHAIN_FILE=cmake-build-debug/build/generators/conan_toolchain.cmake
```

```bash
cd cmake-build-debug
cmake ..
cmake --build . --config Debug
```
