---
title: HelloTriangle
date: 2024-01-17
---

# OpenGL初探——HelloTriangle

本文代码示例来自learnopengl-cn（[你好，三角形](https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/)），通过Opengl完成三角形绘制。

## 1. 引入GLFW和GLAD

- **GLFW** 是一个专门针对OpenGL的C语言库，提供了渲染物体所需的基础接口。它允许用户创建OpenGL上下文、定义窗口参数并处理用户输入。
- **GLAD** 是一个OpenGL加载库，用于在运行时加载OpenGL函数指针。它帮助确定不同显卡和操作系统支持哪些OpenGL函数。

GLFW 可以从其[官方网站下载页](http://www.glfw.org/download.html)获取，支持通过 CMake 编译或直接下载二进制文件。GLAD 通过一个[在线服务](http://glad.dav1d.de/)生成。在GLAD网站上，选择 **C/C++** 语言，选定 OpenGL 版本（本例使用 3.3 以上），模式选择 **Core** 并勾选 **Generate a loader**，然后点击 **生成**（Generate）按钮。

引入时先引入glad，再引入glfw3，以避免编译错误。引入方式见[外部库引用](https://harrysunv9x.github.io/2023/12/14/CorC++/%E5%A4%96%E9%83%A8%E5%BA%93%E5%BC%95%E7%94%A8/)。

```c++
// 包含GLAD和GLFW的头文件
#include <iostream>
#include "./include/glad/glad.h"
#include "./include/GLFW/glfw3.h"
```

## 2. 初始化GLFW

- 初始化GLFW库
- 设置OpenGL版本和模式
- 创建窗口对象
- 设置窗口对象上下文
- 设置窗口大小调整时的回调函数。

```c++
...
void framebuffer_size_callback(GLFWwindow *window, int width, int height);

int main()
{
    // 初始化 GLFW 库
    glfwInit();
    // 配置 GLFW：设置 OpenGL 版本 (3.3)
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    // 使用核心模式
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

    // 创建窗口对象
    GLFWwindow *window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    // 将窗口的上下文设置为当前线程的主上下文
    glfwMakeContextCurrent(window);
    // 设置窗口调整大小时的回调函数
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
	...
}
	
// GLFW: 每当窗口大小改变时，这个回调函数就会被执行
// ---------------------------------------------------------------------------------------------
void framebuffer_size_callback(GLFWwindow *window, int width, int height)
{
    // 确保视口与新窗口的尺寸相匹配
    glViewport(0, 0, width, height);
}
```

## 3. 初始化GLAD

```c++
...
int main(){
	...
	// 初始化 GLAD
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
}
...
```

## 4. 定义三角形顶点数据

```c++
...
int main(){
	...
    // 定义顶点数据
    float vertices[] = {
        -0.5f, -0.5f, 0.0f, // 左下角
        0.5f, -0.5f, 0.0f,  // 右下角
        0.0f, 0.5f, 0.0f    // 顶部
    };
}
...
```

## 5. 编译顶点着色器

- **编写源码**：创建顶点着色器的GLSL源码字符串。
- **创建顶点着色器**：利用 `glCreateShader` 函数创建一个顶点着色器对象。
- **附加源码**：使用 `glShaderSource` 将源码字符串附加到着色器对象上。
- **编译着色器**：调用 `glCompileShader` 编译着色器。
- **检查编译状态**：使用 `glGetShaderiv` 和 `glGetShaderInfoLog` 检查是否编译成功。

```c++
...
int main(){
	...
    // 顶点着色器源码
    const char *vertexShaderSource = "#version 330 core\n"
                                     "layout (location = 0) in vec3 aPos;\n" // 定义了一个名为 aPos 的顶点属性
                                     "void main()\n"
                                     "{\n"
                                     "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n" // 设置最终位置
                                     "}\0";                                                  // 代码结束

    // 创建顶点着色器
    unsigned int vertexShader = glCreateShader(GL_VERTEX_SHADER); // 创建一个顶点着色器对象，返回其 ID
    															  // 参数传递GL_VERTEX_SHADER

    // 将着色器源码附加到着色器对象上
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL); // 将顶点着色器源码附加到顶点着色器对象上
    // 参数解释：
    // vertexShader：顶点着色器对象的 ID，这里用上述创建好的对象
    // 1：指定传递的源码字符串数量
    // &vertexShaderSource：顶点着色器源码数组的地址
    // NULL：源码字符串长度数组，NULL 表示源码字符串以 null 结尾

    // 编译着色器
    glCompileShader(vertexShader); // 编译顶点着色器

    // 检查顶点着色器的编译是否成功
    int success;
    char infoLog[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success); // 获取顶点着色器的编译状态
    // 参数解释：
    // vertexShader：顶点着色器对象的 ID
    // GL_COMPILE_STATUS：获取的参数类型，这里是编译状态
    // &success：存储状态的变量的地址

    if (!success)
    {
        glGetShaderInfoLog(vertexShader, 512, NULL, infoLog); // 获取顶点着色器的编译错误信息
        // 参数解释：
        // vertexShader：顶点着色器对象的 ID
        // 512：信息缓冲区的大小
        // NULL：实际获取的信息长度，NULL 表示不需要这个值
        // infoLog：存储错误信息的字符数组

        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n"
                  << infoLog << std::endl; // 打印错误信息
    }
}
...
```

## 6. 编译片段着色器

与顶点着色器类似。

```c++
...
int main(){
	... 
	// 片段着色器源码
    const char *fragmentShaderSource = "#version 330 core\n"
                                       "out vec4 FragColor;\n"
                                       "void main()\n"
                                       "{\n"
                                       " FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n" // 设置颜色
                                       "}\0";
    // 创建片段着色器
    unsigned int fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    // 将着色器源码附加到着色器对象上
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    // 编译着色器
    glCompileShader(fragmentShader);
    // 检查片段着色器的编译是否成功
    glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
    if (!success)
    {
        glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n"
                  << infoLog << std::endl;
    }
}
...
```

## 7. 创建着色器程序

- **创建程序**：使用 `glCreateProgram` 创建着色器程序。
- **附加着色器**：通过 `glAttachShader` 将顶点和片段着色器附加到程序上。
- **链接程序**：使用 `glLinkProgram` 链接着色器程序。
- **检查链接状态**：使用 `glGetProgramiv` 和 `glGetProgramInfoLog` 检查链接是否成功。
- **删除着色器对象**：链接后，着色器对象不再需要，使用 `glDeleteShader` 删除它们。

```c++
...
int main(){
	... 
    // 创建着色器程序
    unsigned int shaderProgram = glCreateProgram();
    // 将顶点着色器和片段着色器附加到程序上
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    // 链接着色器程序
    glLinkProgram(shaderProgram);
    // 检查着色器程序的链接是否成功
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success)
    {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n"
                  << infoLog << std::endl;
    }
    // 在链接完成后删除着色器对象
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
}
...
```

## 8. 处理VAO和VBO

- **创建VAO和VBO**：使用 `glGenVertexArrays` 和 `glGenBuffers` 创建对象。
- **绑定VAO和VBO**：通过 `glBindVertexArray` 和 `glBindBuffer` 绑定对象。
- **复制顶点数据**：使用 `glBufferData` 将顶点数据复制到VBO。
- **设置顶点属性指针**：配置如何解释顶点数据，使用 `glVertexAttribPointer` 和 `glEnableVertexAttribArray`。

```c++
...
int main(){
	... 
    // 创建 VAO（顶点数组对象）
    unsigned int VAO;
    glGenVertexArrays(1, &VAO);
    // 创建 VBO（顶点缓冲对象）
    unsigned int VBO;
    glGenBuffers(1, &VBO);

    // 绑定 VAO
    glBindVertexArray(VAO);
    // 绑定并设置 VBO
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    
    // 将顶点数据复制到缓冲的内存中
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

    // 设置顶点属性指针
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
    glEnableVertexAttribArray(0);
}
...
```

## 9. 渲染循环

渲染逻辑，添加输入操作。

- **处理输入**：调用 `processInput` 函数处理用户输入。
- **清屏**：使用 `glClearColor` 和 `glClear` 设置清屏颜色并清除颜色缓冲。
- **使用着色器程序**：调用 `glUseProgram` 使用着色器程序。

- **绑定VAO**：使用 `glBindVertexArray` 绑定顶点数组对象。
- **绘制三角形**：通过 `glDrawArrays` 绘制定义好的顶点。
- **交换缓冲区和轮询事件**：使用 `glfwSwapBuffers` 和 `glfwPollEvents` 来更新窗口并处理事件。

```c++
...
void processInput(GLFWwindow *window);

int main(){
	... 
    // 渲染循环
    while (!glfwWindowShouldClose(window))
    {
        // 处理输入
        processInput(window);

        // 清除颜色缓冲区
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);

        // 每次绘制之前使用着色器程序
        glUseProgram(shaderProgram);
        // 每次绘制之前绑定VAO
        glBindVertexArray(VAO);
        // 绘制三角形
        glDrawArrays(GL_TRIANGLES, 0, 3);

        // 交换颜色缓冲（双缓冲模式）
        glfwSwapBuffers(window);
        // 轮询并处理事件
        glfwPollEvents();
    }
}
...
// 处理所有输入: 查询 GLFW 是否有相关键被按下/释放，并做出相应反应
// ---------------------------------------------------------------------------------------------------------
void processInput(GLFWwindow *window)
{
    // 如果按下了Escape键，则设置窗口应该关闭
    if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}
```

在 `while` 循环中，每次绘制之前使用着色器和绑定VAO是OpenGL的标准实践：

1. **状态机机制**：OpenGL按照当前状态执行操作，因此必须指定每次渲染所用的着色器和VAO。
2. **多物体渲染**：不同物体可能需要不同的着色器和顶点配置，因此每次渲染前的绑定确保了正确的配置。
3. **代码清晰和扩展性**：这种做法使代码更易于理解和维护，特别是在添加新物体或着色器时。
4. **防止状态污染**：每次绘制前的重新绑定避免了之前渲染步骤的设置影响当前绘制。

## 10. 清理并退出

```c++
...
int main(){
	... 
    // 退出前释放资源
    glfwTerminate();
    glDeleteVertexArrays(1, &VAO);
    glDeleteBuffers(1, &VBO);
    glDeleteProgram(shaderProgram);

    return 0;
}
...
```

