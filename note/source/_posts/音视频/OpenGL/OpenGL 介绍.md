---
title: OpenGL 介绍
date: 2025-11-19 16:51:56
categories: 音视频
tags: OpenGL
---

## 🧭什么是 OpenGL？

**OpenGL（Open Graphics Library，开放图形库）**是一个跨平台、跨语言的**图形 API 标准**，用于在各种系统上绘制 2D 和 3D 图形。

它不是一个程序、库或驱动，而是由**Khronos Group** 制定的一份**规范（Specification）**。

这份规范只定义了：

- 每个函数的名称、参数与返回值、调用行为；
- 错误处理规则；
- 渲染管线各阶段应如何工作。

但是，**它并没有任何具体实现**。真正的实现由**显卡厂商（NVIDIA、AMD、Intel 等）**在各自的 GPU 驱动中完成。

### 🔹OpenGL 的规范层（Specification）

由 Khronos Group 维护：

- 定义 OpenGL 应该具备的功能；
- 规定函数调用的语义；
- 制定每个版本的功能集合与行为。

例如：

```cpp
void glClearColor(GLfloat red, GLfloat green, GLfloat blue, GLfloat alpha);
```

规范定义：

> 该函数设置清除颜色缓冲区时使用的颜色值。不规定怎么实现，只规定效果。

------

### 🔹OpenGL 的实现层（Implementation）

OpenGL 的真正执行代码存在于 **GPU 驱动程序** 中。当你调用 OpenGL API 时，最终会流向：

| 平台    | OpenGL 加载库    | 实现方                |
| ------- | ---------------- | --------------------- |
| Windows | opengl32.dll     | 显卡厂商的驱动（ICD） |
| Linux   | libGL.so         | Mesa / NVIDIA / AMD   |
| macOS   | OpenGL.framework | Apple（已停更）       |

这些驱动会：

- 实现所有 OpenGL 函数；
- 将命令翻译为 GPU 的底层指令；
- 管理显存、纹理、缓冲区等资源；
- 负责与系统窗口系统交互（如 WGL、GLX、EGL）。

------

## 🧰 GLAD 加载库的作用

既然每个 OpenGL 实现由不同厂商提供，而每个系统支持的 OpenGL 版本又不一样，程序运行时必须「动态加载」这些函数。

> 例如，OpenGL 4.6 有几百个函数，操作系统默认库只支持到 1.1（Windows）。

所以我们需要 **GLAD** 这样的「函数加载器（Loader）」来帮我们：

- 查询系统支持的 OpenGL 版本；
- 动态加载所有函数地址；
- 为每个函数生成函数指针；
- 提供可直接调用的接口。

因此，GLAD 并不是 OpenGL 的一部分，它只是帮你「找到」真正的 OpenGL 函数。

------

## 🎨 OpenGL 的定位与功能

OpenGL 是一个**图形渲染 API**，目标是提供一种与平台无关的方式来访问 GPU 渲染管线。

它能做的事包括：

- 绘制点、线、三角形（所有图形的基本单元）；
- 管理顶点缓冲、纹理、着色器；
- 执行几何变换、光照、混合等图形运算；
- 通过 GLSL（OpenGL Shading Language）语言编写 GPU 着色器。

不能做的事包括：

- 创建窗口、接收键盘鼠标输入（这由 GLFW / SDL / Qt 等库完成）；
- 播放视频或音频；
- 执行物理模拟。

------

## 🧱 OpenGL 的版本演化

| 版本                    | 时间            | 主要特性                                                   |
| ----------------------- | --------------- | ---------------------------------------------------------- |
| **1.x (1992–2003)**     | SGI 发布        | 固定管线，函数式 API（如 `glBegin/glEnd`）                 |
| **2.0 (2004)**          |                 | 引入 GLSL 着色语言，开始支持可编程管线                     |
| **3.0 (2008)**          |                 | 废弃旧式固定管线，现代化设计                               |
| **3.3 (2010)**          |                 | 成为最常用的现代兼容版本（被广泛支持）                     |
| **4.0–4.6 (2010–2017)** |                 | 支持计算着色器、延迟渲染、多重采样、Direct State Access 等 |
| **当前最新版本**        | **4.6（2017）** | Khronos Group 最新正式版本                                 |

> 💡 OpenGL 4.6 仍然是目前广泛使用的主流版本。Khronos Group 现在的重心已经转向了 **Vulkan**。

------

## 🧮 OpenGL 的工作原理（概览）

当你调用 OpenGL 函数时，大致流程如下：

```
CPU 程序
   ↓
OpenGL API (glXXX)
   ↓
OpenGL 驱动（厂商实现）
   ↓
命令缓冲区（Command Buffer）
   ↓
GPU 渲染管线
      ├─ 顶点着色器（Vertex Shader）
      ├─ 图元装配（Primitive Assembly）
      ├─ 光栅化（Rasterization）
      ├─ 片段着色器（Fragment Shader）
      └─ 帧缓冲（Framebuffer）
   ↓
显示器输出像素
```

> OpenGL 的强大之处在于，它让你用统一的 API 操控 GPU 渲染管线的每个阶段。

------

## 🌍 OpenGL 的优点

1. **跨平台**：Windows / macOS / Linux / Android 都可用。
2. **跨厂商**：NVIDIA / AMD / Intel 都支持。
3. **跨语言**：C/C++、Python、Rust、Java 都可通过绑定使用。
4. **成熟稳定**：已有 30+ 年历史，文档与教程极其丰富。
5. **硬件加速**：几乎所有 GPU 都支持，性能优越。
6. **实时渲染**：支持游戏、可视化、仿真、CAD、科学绘图等场景。

------

## ⚠️ OpenGL 的局限与衰退

1. **驱动层黑箱**：厂商实现差异大，bug 难排查；
2. **性能瓶颈**：驱动封装层较厚，不如 Vulkan / DX12 低延迟；
3. **线程不友好**：OpenGL Context 通常限于单线程；
4. **API 设计老旧**：部分函数保留兼容性导致混乱；
5. **macOS 停止更新**：Apple 已弃用 OpenGL，转向 Metal。

