---
title: 标准IO函数——popen
date: 2023-01-22
---

## 标准IO函数——popen

## 用法

如果想在程序运行的过程中执行某个命令行，并且与当前的程序不冲突，可以通过popen创建一个新的进程来执行特定的命令，并且允许程序与这个进程进行交互。

```c++
FILE *popen(const char *command, const char *mode);			//原型
                                                   			//command: 要执行的命令
                                                   			//mode: 指定通信模式，可以是 "r"（读）或 "w"（写）
```

## 代码示例

Python是一个实时的解释器，每次输入都会返回一个输出显示。是一个很好的命令好交互示例。

### 代码

```c++
#include <stdio.h>
#include <iostream>
#include <string>
#include <thread>
int main()
{
    FILE *pythonPipe = popen("python -i", "w"); // 初始化命令与模式。读模式r，写模式w。
    if (!pythonPipe)
    { // 不要忘记错误处理
        perror("初始化失败");
        return 1;
    }

    std::string pythonSentence;
    bool isRunning = 1;
    while (isRunning)
    {
        std::getline(std::cin, pythonSentence); // 输入，这里不要用std cin >> pythonSentence，因为无法获取空白符
        pythonSentence += "\n";                 // 注意，python是行缓冲，要加入换行符才可以及时回显

        fprintf(pythonPipe, pythonSentence.c_str());
        fflush(pythonPipe);                     // 输入并刷新缓冲区

        if (pythonSentence == "exit()\n")
        {
            isRunning = false;                  // 停止标识符
        }
    }

    pclose(pythonPipe);
    return 0;
}
```

### 相关函数

1. **popen**

   - **功能**: 创建一个管道，启动一个新进程，并连接该进程的标准输入或输出到管道。

   - **用法**: 

     ```
     FILE *popen(const char *command, const char *mode);
     ```

     - `command`: 要执行的命令。
     - `mode`: 打开管道的模式，`"r"`（读模式）或 `"w"`（写模式）。

2. **perror**

   - **功能**: 打当系统调用失败时，通常会设置全局变量 `errno`。`perror` 根据 `errno` 的值打印出相应的错误消息。

   - **用法**: 

     ```
     void perror(const char *s);
     ```

     - `s`: 要打印的消息前缀。

3. **fprintf**

   - **功能**: 格式化输出函数，用于向指定的文件或流输出格式化文本。

   - **用法**: 

     ```
     int fprintf(FILE *stream, const char *format, ...);
     ```

     - `stream`: 指向 `FILE` 对象的指针，指定输出目标。
     - `format`: 格式化字符串。

4. **fputs**

   - **功能**: 向指定的文件或流写入一个字符串。

   - **用法**: 

     ```
     int fputs(const char *s, FILE *stream);
     ```

     - `s`: 要写入的字符串。
     - `stream`: 目标文件或流。

5. **fflush**

   - **功能**: 清除文件流的输出缓冲区，将缓冲区中的所有未写数据写入到基础输出设备。

   - **用法**: 

     ```
     int fflush(FILE *stream);
     ```

     - `stream`: 目标文件或流。

6. **pclose**

   - **功能**: 关闭 `popen` 打开的流。

   - **用法**: 

     ```
     int pclose(FILE *stream);
     ```

     - `stream`: 由 `popen` 返回的流。