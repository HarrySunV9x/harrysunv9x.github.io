---
title: CMake(Mingw示例)
date: 2023-12-15
---

# CMake(Mingw示例)

## 背景

操作系统（OS）的底层实现各异，导致了不同OS间可执行文件的兼容性问题。这种问题在Linux环境中尤为突出，因为Linux不仅有多种发行版，还有众多版本，即使是微小的版本更新也可能导致应用程序不兼容。这通常要求通过重新编译源码来解决。

在不同操作系统中遇到可执行文件不兼容的问题，着实令人困扰。笔者第一年的工作就是处理网卡兼容性，一款网卡可能需要在数十个操作系统版本上进行测试。在明显与操作系统无关的问题出现时，由于系统更新导致的可执行文件不兼容令人十分头疼。

为解决这一问题，一种有效的方案是跨平台编译。CMake是一种实现跨平台编译的构建系统，可以生成不同平台的编译脚本。Mellanox网卡驱动和笔者目前正在学习的OpenGL中使用的GLFW库都采用了CMake，使用起来非常便捷。

当然，CMake只是众多跨平台编译方法中的一种。例如，知名的DPDK就没有使用CMake，而是采用了meson。这里只是简单提及，不作深入讨论。

## 步骤

CMake工程的一般步骤如下：

1. **编写源文件**

   创建源代码文件，例如 `.c`、`.cpp` 或 `.h` 文件。

2. **编写 CMakeLists.txt**

   在编译时，我们主要解决以下这些问题：

   1. **编译器选择**：根据您的代码是用 C 还是 C++ 编写的来选择编译器，如 GCC、Clang 或 MSVC。
   2. **编译版本**：指定编译标准，如 C++11、C++14、C++17 等，这影响可用的语言特性和库。
   3. **编译选项**：设置编译选项，如优化级别、是否将警告视为错误等，这些选项有助于提高代码质量和性能。
   4. **源文件**：确定哪些源文件将被编译，这可能涉及到在项目中包含或排除特定文件或模块。
   5. **外部库引用**：管理对外部库如 GLFW、Boost 等的引用，包括定位库文件和指定链接选项。
   6. **其他配置**：
      - **测试**：配置单元测试和集成测试。
      - **平台特定配置**：如对 Android、Windows 等操作系统的特殊处理。
      - **打包和分发**：设置应用程序的打包方式及分发所需的文件和依赖项。
      - **自定义宏**：定义预处理器宏，用于条件编译或简化代码。
   7. **优化**：除了编译选项中的优化级别，还可能涉及更具体的性能调优，如内存管理、并行处理等。

   CMake 将这些问题都放在 CMakeLists.txt 文件里，供我们进行配置。

3. **生成 Makefile**

4. **make**

## 示例

通过Socket编程的示例，展示CMake的使用：

1. 编写源文件

   **main.cpp**

   ```c++
   #include <winsock2.h>				//引用系统库文件
   #include "include/DemoSocket.h"		//引用自定义头文件
   
   int main(int argc, char *argv[])
   {
   	//实现逻辑
   	...
   	
       return 0;
   }
   ```

   DemoSocket.h

   ```C++
   // Socket类定义，提供基本的网络通信接口
   class DemoSocket
   {
       // 接口声明
       ...
   }
   ```

   DemoSocket.cpp

   ```c++
   // Socket类实现
   // 接口实现
   DemoSocket::DemoSocket(){
       ...
   }
   ...
   ```

2. 编写CMakeList.txt

   ```cmake
   # 指定CMake的最低版本要求
   cmake_minimum_required(VERSION 3.10)
   
   # 明确指定C和C++编译器的路径
   # 注意：在多平台项目中硬编码路径可能导致可移植性问题
   SET(CMAKE_C_COMPILER "C:/msys64/mingw64/bin/gcc.exe")
   SET(CMAKE_CXX_COMPILER "C:/msys64/mingw64/bin/g++.exe")
   
   # 设置项目名称，这里项目名称为 'DemoSocket'
   project(DemoSocket)
   
   # 设置C++标准为C++11，并确保其为必需
   # 这意味着如果编译器不支持C++11标准，则会报错
   set(CMAKE_CXX_STANDARD 11)
   set(CMAKE_CXX_STANDARD_REQUIRED True)
   
   # 添加可执行文件的源文件
   # 这里添加了两个源文件 'Main.cpp' 和 'DemoSocket.cpp'
   # CMake会自动处理相关的头文件依赖
   add_executable(DemoSocket Main.cpp DemoSocket.cpp)
   
   # 将外部库链接到项目
   # 这里链接了 'wsock32' 库，通常用于Windows平台的Socket编程
   target_link_libraries(DemoSocket wsock32)
   
   ```

3. 生成Makefile

   ```shell
   cmake -G "MinGW Makefiles" .. # 这里 -G 指定生成器，注意后面有.. 表示项目目录，使用这条命令在当前目录生成 Makefile
   ```

4. make

   ```shell
   mingw32-make.exe # mingw的编译工具
   ```

由于Windows系统的特殊性，这里描述了Mingw的命令使用方法（就两条命令坑了我一下午）。在Linux系统里实现会更简单：

```shell
cmake			#如果没有就装一下：apt-get install cmake 或者 yum install cmake
make
make install
```

