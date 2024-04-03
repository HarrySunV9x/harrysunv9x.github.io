---
title: CMakeList
data: 2024-03-30
---

**设置cmake版本与项目**

```cmake
cmake_minimum_required(VERSION 3.14)  	# for add_link_options and implicit target directories.
project("llama.cpp" C CXX)
include(CheckIncludeFileCXX)			# 检查是否包含头文件
```

**编译配置**

```cmake
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)	# 编译过程生成 compile_commands.json
```

能够有效提高一些工具的代码跳转、补全等功能。

**设置构建类型**

```cmake
if (NOT XCODE AND NOT MSVC AND NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()
```

`set(CMAKE_BUILD_TYPE Release)`：将`CMAKE_BUILD_TYPE`设置为"Release"。

`CACHE STRING "Build type"`：将构建类型设置为一个缓存变量，并提供了一个描述性的字符串。





如果没有`XCODE`、`MSVC`和`CMAKE_BUILD_TYPE`配置，则使用默认的构建配置构建类型（Build Type）：默认Release，可选Debug、Release、MinSizeRel、RelWithDebInfo。

CMake提供了几种默认的构建类型：

- Debug：没有优化并附带完整的调试信息，通常在开发和调试过程中使用。
- Release：没有调试信息，提供全面的速度优化。
- RelWithDebInfo：在某种程度上是前两者的妥协，它的目的是使性能接近于发布版，但仍允许某种程度的调试。通常会对速度进行大部分优化，但也会启用大部分调试功能。默认禁用断言。
- MinSizeRel：通常只用于有限的资源环境，如嵌入式设备。代码是针对大小而不是速度进行优化的，并且不创建调试信息。

**变量赋值**

```cmake
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
```

```cmake
if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(LLAMA_STANDALONE ON)

    # configure project version
    # TODO
else()
    set(LLAMA_STANDALONE OFF)
endif()
```

```cmake
if (EMSCRIPTEN)
    set(BUILD_SHARED_LIBS_DEFAULT OFF)

    option(LLAMA_WASM_SINGLE_FILE "llama: embed WASM inside the generated llama.js" ON)
else()
    if (MINGW)
        set(BUILD_SHARED_LIBS_DEFAULT OFF)
    else()
        set(BUILD_SHARED_LIBS_DEFAULT ON)
    endif()
endif()
```

```cmake
#
# Option list
#

if (APPLE)
    set(LLAMA_METAL_DEFAULT ON)
else()
    set(LLAMA_METAL_DEFAULT OFF)
endif()
```