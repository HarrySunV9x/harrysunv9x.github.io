---
title: Socket编程
date: 2023-12-15
---
# Socket编程

## 背景

在我们日常生活中，进程间通信的应用无处不在。想象一下，某天你在微信上向朋友抱怨工作压力巨大，头都秃了。然后你打开了小红书，准备放松一下，结果映入眼帘一个醒目的开屏广告：“编程天才，也可以是秀发健康专家。”

这种大数据驱动的精准广告推荐，可以通过分析你在手机聊天软件、输入法、音频设备等应用中的数据来实现。它涉及到进程间的通信，将你可能感兴趣的信息从某个应用拿到小红书中，从而实现个性化的广告推送。

**进程通信，简单说就是让不同的计算机程序之间能够交流和分享信息的方法。**就好比人们之间可以通过电话、短信或互联网来交流一样，计算机程序之间也需要一种方式来沟通。进程通信的方式有很多种，光考试也对这些概念看到很多次了：管道（pipe）、命名管道（FIFO）：、消息队列（message queue）、共享内存（shared memory）、信号量（semaphore）、套接字（socket）。

如果想在不同设备之间进行通信，比如发送微信消息给朋友、在淘宝下单购物、或回复工作伙伴的电子邮件等，通常都需要使用网络来实现。在各种进程通信方式中，**套接字** 是唯一能够实现不同设备之间通信的方式。

本文将详细介绍Socket编程，并用实例展示Socket编程在Windows与Android(Linux)平台的实现。

## 概念

- **定义**: Socket（套接字）是网络通信的端点，你可以把它想象成电话通话中的电话机。当两台计算机通过网络通信时，它们各自有一个 Socket。
- **功能**: 它负责建立、维护、终止网络上的会话，并通过这个会话进行数据的发送和接收。

## 类型

- **流式 Socket (TCP)**: 提供面向连接、可靠的数据流服务。使用 TCP（传输控制协议）保证数据完整性和顺序性，适用于要求高可靠性的应用，如网页服务器。
- **数据报 Socket (UDP)**: 提供无连接的服务，发送和接收数据报文。使用 UDP（用户数据报协议），适用于实时应用，如在线游戏

## 步骤

以下表格同步了服务端与客户端套接字的流程：

| 服务端（接收）                                               |                     客户端（发送）                     |
| :----------------------------------------------------------- | :----------------------------------------------------: |
| **创建 Socket**: 创建服务端套接字，指定使用 TCP 或 UDP。     |         **创建 Socket**: 指定使用 TCP 或 UDP。         |
| **绑定 Socket**: 将创建的 Socket 绑定到特定的 IP 地址和端口上。 |                                                        |
| **监听连接**: 对于 TCP，监听来自客户端的连接请求。           |                                                        |
| **接受连接**: 创建客户端套接字，接受客户端的连接。           | **建立连接**: 对于 TCP，连接到服务器的 IP 地址和端口。 |
| **数据交换**: 接收和发送数据。                               |       **数据交换**: 发送请求并接收服务器的响应。       |
| **关闭 Socket**: 结束通信，关闭客户端套接字，等待其他连接。  |         **关闭 Socket**: 结束通信，关闭连接。          |

## 实现

### 单Socket方案

以传输文件（TCP）为例，介绍Socket的实现。

#### Windows端

##### 服务端

引入头文件：

```c++
#include <iostream>
#include <winsock2.h> //注意这个，Windows需要引入该文件
#include <sstream>
#include <fstream>
#include <string>
```

开启wsaData: 

```C++
WSADATA wsaData; // 声明wsa数据，Windows需要
if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
{ // 传入版本、数据，启动wsaData
    std::cout << "WSA Start Up Error!" << std::endl;
    return 1;
}
```

创建套接字(socket)：

```C++
// 创建套接字。PF_INET代表IPv4协议族，SOCK_STREAM表示TCP连接，IPPROTO_TCP指定使用TCP协议
SOCKET linkSocket = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP);
if (linkSocket == INVALID_SOCKET)
{
    errorMessage("Create");
    return;
}
```

绑定套接字(bind)：

```C++
memset(&sockAddr, 0, sizeof(sockAddr));                // 确保sockAddr结构被清零。memset用于填充内存块，这里用0填充
sockAddr.sin_family = PF_INET;                         // 设置套接字地址的家族为IPv4
sockAddr.sin_addr.s_addr = inet_addr(DEFAULT_ADDRESS); // 设置IP地址。inet_addr将十进制的IP地址转换为网络字节顺序
sockAddr.sin_port = htons(DEFAULT_PORT);               // 设置端口号。htons确保端口号的字节顺序符合网络标准
if (bind(linkSocket, (SOCKADDR *)&sockAddr, sizeof(SOCKADDR)) == SOCKET_ERROR)
{
    errorMessage("Bind");
    closesocket(linkSocket);
    return;
}
```

监听套接字(listen)：

```C++
// 监听套接字，设置最大等待连接数
if (listen(linkSocket, 20) == SOCKET_ERROR)
{
    errorMessage("Listen");
    closesocket(linkSocket);
    return;
}
```

接受连接，创建客户端套接字建立连接(accept)：

```C++
SOCKADDR clntAddr;
int nSize = sizeof(SOCKADDR);
SOCKET clntSock = accept(linkSocket, (SOCKADDR *)&clntAddr, &nSize);
if (clntSock == INVALID_SOCKET)
{
    errorMessage("Accept");
    closesocket(linkSocket);
    return;
}
```

数据交换，接收文件名，接收文件(recv)：

```C++
// 接收文件名
char fileNameBuffer[260]; // 假定文件名不会超过260个字符
int fileNameLength = recv(clntSock, fileNameBuffer, 260, 0);
if (fileNameLength <= 0)
{
    std::cerr << "Failed to receive file name." << std::endl;
    closesocket(clntSock);
    return;
}
// 文件接收和写入
std::string receivedFileName = fileNameBuffer;
std::ofstream outfile(receivedFileName, std::ofstream::binary);
if (!outfile)
{
std::cerr << "Error: Cannot open 'received_file.txt' for writing." << std::endl;
closesocket(clntSock);
return;
}
// 从socket读取数据并写入文件
int valread;
char buffer[1024] = {0};
while ((valread = recv(clntSock, buffer, 1024, 0)) > 0)
{
    outfile.write(buffer, valread);
}
// 读取socket时的错误处理
if (valread == SOCKET_ERROR)
{
    errorMessage("Receive");
    outfile.close();
    closesocket(clntSock);
    return;
}
// 数据传输后关闭文件
outfile.close();
```

结束通信，关闭客户端套接字，等待其他连接：

```C++
closesocket(clntSock);
```

##### 客户端

初始化文件选择对话框

```C++
OPENFILENAME ofn; // Windows文件选择对话框结构
char szFile[260]; // 存储选中的文件名
    
// 初始化文件选择对话框
ZeroMemory(&ofn, sizeof(ofn));
ofn.lStructSize = sizeof(ofn);
ofn.hwndOwner = NULL;
ofn.lpstrFile = szFile;
ofn.lpstrFile[0] = '\0';
ofn.nMaxFile = sizeof(szFile);
ofn.lpstrFilter = "All\0*.*\0Text\0*.TXT\0";
ofn.nFilterIndex = 1;
ofn.lpstrFileTitle = NULL;
ofn.nMaxFileTitle = 0;
ofn.lpstrInitialDir = NULL;
ofn.Flags = OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST;
```

建立连接：

```
// 连接到服务器
if (connect(linkSocket, (SOCKADDR *)&sockAddr, sizeof(SOCKADDR)) == SOCKET_ERROR)
{
    errorMessage("Connect");
    closesocket(linkSocket);
    return;
}
```

数据交换：

```C++
// 打开文件选择对话框并获取文件名
if (GetOpenFileName(&ofn) == false)
{
    // 使用ofn.lpstrFile作为文件路径
    std::cerr << "File Open Error: File not found or cannot be opened." << std::endl;
    closesocket(linkSocket);
    return;
}
// 发送文件名到服务器
std::string fileName = ofn.lpstrFile; // 获取完整文件路径
auto lastSlash = fileName.find_last_of("\\/");
if (lastSlash != std::string::npos)
{
    fileName = fileName.substr(lastSlash + 1); // 提取文件名
}
if (send(linkSocket, fileName.c_str(), fileName.size() + 1, 0) == SOCKET_ERROR)
{ // 发送文件名
    errorMessage("Send");
    closesocket(linkSocket);
    return;
}
Sleep(100); // 稍微等待，确保文件名被正确接收

// 打开并读取文件
std::ifstream infile(ofn.lpstrFile, std::ifstream::binary);
if (!infile)
{
    DWORD error = GetLastError();
    std::cerr << "File Open Error: " << error << std::endl;
    return; // 退出函数
}
// 文件传输逻辑
infile.seekg(0, infile.end);     // 设置输入文件流的读取位置到文件末尾
long totalSize = infile.tellg(); // 获取文件的总大小
infile.seekg(0, infile.beg);     // 重新设置文件流的读取位置到文件开始
long totalSent = 0;              // 初始化已发送的数据总量为0
// 循环读取文件并发送数据
char buffer[1024] = {0};
while (true)
{
    // 从文件中读取数据到缓冲区
    infile.read(buffer, sizeof(buffer));
    // 获取实际读取的字节数
    std::streamsize bytes_read = infile.gcount();
    // 如果读取到数据
    if (bytes_read > 0)
    {
        // 通过套接字发送数据
        if (send(linkSocket, buffer, static_cast<int>(bytes_read), 0) == SOCKET_ERROR)
        {
            errorMessage("Send failed");
            infile.close();
            closesocket(linkSocket);
            return;
        }
        // 累加已发送的数据量
        totalSent += bytes_read;
        // 计算并显示发送进度
        double progress = static_cast<double>(totalSent) / totalSize * 100;
        std::cout << "\rProgress: " << std::fixed << std::setprecision(2) << progress << "%" << std::flush;
    }

    // 检查是否到达文件末尾
    if (infile.eof())
    {
        // 到达文件末尾，结束循环
        break;
    }
    // 检查文件读取是否失败
    if (infile.fail())
    {
        // 如果读取失败，输出错误信息并返回
        std::cerr << "Failed to read from file." << std::endl;
        infile.close();
        closesocket(linkSocket);
        return;
    }
}
std::cout << std::endl; // 进度条换行
// 关闭文件和socket
infile.close();
shutdown(linkSocket, SD_SEND);
```

接收服务器回传消息，关闭Socket：

```C++
// 接收服务器传回的数据
char szBuffer[MAXBYTE] = {0};
int ret = recv(linkSocket, szBuffer, MAXBYTE, NULL);
// 输出接收到的数据
if (ret == SOCKET_ERROR)
{ // 当返回值为-1，代表连接失败进行错误处理
    errorMessage("Recive");
}
else if (ret == 0)
{ // TCP 协议确保数据的可靠传输。在实际应用中，接收到长度为零的数据包通常被视为连接终止的信号。
    // 根据本次应用的场景，对于Windows server来说，在socket客户端场景下这是不正常的。
    // 而Android（Linux）服务器场景，文件在接收完后会在finish帧后直接返回一个ack帧关闭连接，因此可能是正常的
    // 具体原理暂时不过多研究，有时间可考虑在stackoverfloww上提问下~
    // 根据这个现象要注意，Android（Linux）场景下，一次socket只能完成一种行为，无法先接收再确认......
    std::cout << "Socket Recive 0: If your server is not windows, this copy may finished!" << std::endl;
}
else
{
    std::cout << "Message from server: " << szBuffer << std::endl;
}
```

#### Android（Linux）端

接收端代码在大部分方面保持一致，唯一不同的是接口不同。在这里，只涉及到服务端的接收功能。我们不再分段，直接提供完整的代码段：

```C++
#include <stdio.h>          // 用于标准输入输出函数，如 printf, scanf
#include <string.h>         // 提供字符串操作函数，如 strcpy, strlen
#include <stdlib.h>         // 包含常用的库函数，如 malloc, free
#include <unistd.h>         // 提供UNIX标准系统调用的接口，如 read, write, close
#include <arpa/inet.h>      // 用于网络地址转换函数，如 inet_pton, inet_ntop
#include <sys/socket.h>     // 提供套接字接口函数和数据结构，如 socket, connect
#include <netinet/in.h>     // 定义IP地址和端口号的结构，如 sockaddr_in
#include <iostream>         // 用于C++标准输入输出流，如 std::cout, std::cin
#include <android/log.h>    // 提供Android日志输出函数，如 __android_log_print
#include <sys/types.h>      // 定义用于系统调用的数据类型，如 pid_t, off_t
#include <cstring>          // 提供C风格字符串处理函数，如 strcpy, strlen
#include <fstream>          // 用于C++文件流操作，如 std::ifstream, std::ofstream

__android_log_print(ANDROID_LOG_INFO, "Demo", "Starting DemoSocket");
//创建套接字
serv_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
if (serv_sock == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Socket creation failed: %s",
                        strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "Demo", "Socket created successfully");
// 将套接字和IP、端口绑定
memset(&serv_addr, 0, sizeof(serv_addr));  // 清空地址结构
serv_addr.sin_family = AF_INET;            // 使用IPv4地址
serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  // 设置IP地址
serv_addr.sin_port = htons(50000);         // 设置端口号
    // 绑定套接字
if (bind(serv_sock, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Bind failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "Demo", "Bind successful");
// 监听套接字，设置最大等待连接数
if (listen(serv_sock, 20) == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Listen failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "Demo", "Listening");
// 接受客户端连接请求
struct sockaddr_in clnt_addr;
socklen_t clnt_addr_size = sizeof(clnt_addr);
int clnt_sock = accept(serv_sock, (struct sockaddr *) &clnt_addr, &clnt_addr_size);
if (clnt_sock == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Accept failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "Demo", "Client connected");
// 接收文件名
char fileNameBuffer[260];
int fileNameLength = recv(clnt_sock, fileNameBuffer, 260, 0);
if (fileNameLength <= 0) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Failed to receive file name: %s",
                        strerror(errno));
    close(clnt_sock);
    return;
}
std::string receivedFileName = fileNameBuffer;
__android_log_print(ANDROID_LOG_INFO, "Demo", "Received file name");
// 构造文件的完整路径
std::string internalPath = "/data/data/com.harry.apk/";
std::string fullPath = internalPath + receivedFileName;
std::ofstream outfile(fullPath, std::ofstream::binary);
// 检查文件是否成功打开
if (!outfile) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Cannot open file: %s",
                        fullPath.c_str());
    close(clnt_sock);
    return;
}
// 读取数据并写入文件
int valread;
char buffer[1024] = {0};
while ((valread = recv(clnt_sock, buffer, 1024, 0)) > 0) {
    outfile.write(buffer, valread);
}
// 检查数据接收是否有错误
if (valread < 0) {
    __android_log_print(ANDROID_LOG_ERROR, "Demo", "Error receiving data: %s",
                        strerror(errno));
    outfile.close();
    close(clnt_sock);
    return;
}
__android_log_print(ANDROID_LOG_INFO, "Demo", "File transfer completed");
// 关闭文件
outfile.close();
__android_log_print(ANDROID_LOG_INFO, "Demo", "File closed");
// 发送接收确认消息
const char *RECEIPT_CONFIRMATION = "RECEIPT_CONFIRMED";
send(clnt_sock, RECEIPT_CONFIRMATION, strlen(RECEIPT_CONFIRMATION) + 1, 0);
// 关闭客户端套接字
close(clnt_sock);
__android_log_print(ANDROID_LOG_INFO, "Demo", "Client socket closed");
// 关闭套接字
close(serv_sock);
__android_log_print(ANDROID_LOG_INFO, "Demo", "Socket closed");
```

Main函数（Android端）：

需要注意的是，在Android平台，我们需要通过JNI接口实现，而且需要多线程：

```C++
#include <jni.h>
#include <string>
#include <thread>

#include "DemoSocket.h"

void startSocketOperation() {
    DemoSocket receiveFileSocket;
    receiveFileSocket.ReceiveSocket();
}

extern "C" JNIEXPORT void JNICALL
Java_com_harry_apk_MainActivity_socketButtonFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    __android_log_print(ANDROID_LOG_INFO, "Demo", "DemoSocket init...");
    std::thread socketThread(startSocketOperation);
    socketThread.detach(); // 使线程独立执行
}
```

### 多Socket连接方案

上述代码有两个很明显的问题：

1、单Socket连接：很明显，在服务端接收到文件回送确认消息后，服务端就关闭了，这种实现过程过于简单了，无法满足日常需要。我们可以通过添加一个while循环解决单次传输的问题（见下面代码的mode 1）。

2、阻塞：如果有多个socket连接同时过来会发生什么呢？是的，会发生排队现象——只有一个Socket结束了，另一个Socket才会继续。这会导致硬件资源的浪费。

#### 多线程

我们可以通过多线程解决这个问题：

```C++
// .h头文件，将accept与业务功能抽象出来做封装，这里省略实现了，上面的代码改改就可以
/**
 * 创建ClntSocket
 * @return SOCKET: 返回创建的Socket
 */
SOCKET createClntSocket() const;

/**
* Socket业务功能：接收文件
* @param receive_socket 接收文件的Socket
*/
static void ReceiveFile(SOCKET receive_socket);

// .cpp文件， socket功能的视线，这里在类初始化时已经完成了socket的创建，下一步是accept或者connect
void AnkiSocket::startSocket(char *command) {
    if (strcmp(command, "server") == 0) {
        std::cout << "Please select socket link mode: " << std::endl;
        std::cout << "1. Single-threaded Mode" << std::endl;
        std::cout << "2. Multi-threaded Mode " << std::endl;
        int mode;
        std::cin >> mode;
        if (mode != 1 && mode != 2) {
            std::cout << "Unrecognized Mode";
            return;
        }
        serverSocket();
        if (mode == 1) {
            while(1){
                SOCKET SingleSocket = createClntSocket();
                //可选择业务
                ReceiveFile(SingleSocket);
            }
        }
        if (mode == 2) {
            while(1){
                // 启用多线程，实现多个Socket建立
                SOCKET newClntSocket = createClntSocket();
                std::thread newClntSocketThread(ReceiveFile, newClntSocket);
                newClntSocketThread.detach();
            }
        }
    } else {
        std::cout << "Please select your file. " << std::endl;
        clientSocket();
    }
}
```

多线程编程在多种应用场景中都是优秀的解决方案，比如GUI程序、实时系统、并发任务处理等，网络编程（Socket）也是其中之一。

但是Socket的多线程也有无法忽视的问题：

然而，Socket编程中的多线程处理同样面临着一些不容忽视的挑战：

1. **资源浪费**：每个线程都会消耗系统资源，特别是在处理大量并发连接时，线程数量的增加可能导致资源的过度消耗。
2. **性能损耗**：线程间的上下文切换可能导致性能损耗，尤其是在高负载情况下。
3. **数量限制**：操作系统通常对可创建的线程数量有限制，这可能影响到程序的扩展性和稳定性。

这些问题会随着Socket数量的增多而变得更加明显，比如，一个服务器连接数百万用户，这种性能损耗该有多大呢？

当然，我们也可以采用如连接池、锁、线程安全机制等多线程处理技术来应对这些问题。但无法避免的，这些手段会让代码变的更庞大、复杂。

那么有没有一种方案，他既可以单线程，又可以管理多个Socket连接呢？

I/O多路复用技术来了~

#### I/O多路复用

I/O多路复用，是用户态与内核态相互配合，从而实现对Socket连接的管理。

首先，要明确一下：**I/O多路复用是异步方案，不是并行方案；I/O多路复用是单线程的，它无法替代多线程。**

异步和并行的区别：

假设我们有两个工作线A、B，异步的概念是：

当任务A遇到一个需要等待的操作时，为了避免硬件资源浪费，将控制权返回，去调用任务B。而任务B卡主时，再返回任务A。异步的执行是线性的。

并行则很好理解，是A、B两个线路同时操作。
![image-20240228153938312](.\image\异步与并行socket.png)

如图是异步与并行Socket的区别：

- 异步仍是单线程，在遇到如Listen、recv、send等可能会阻塞的操作时，会切换到另一个Socket任务继续执行；并行在本任务按顺序执行。
- 异步是上层应用与内核态应用共同协作实现的操作；并行是操作系统在CPU上启动一个线程执行。
- 异步原子操作无法分割，如send执行时，无法中断去执行另一个任务；并行各执行各的，互不打扰。

##### Select

**普通Socket的原理**

Socket的recv一般原理是这样的：

1. **创建Socket并获取文件描述符**：
   - 在用户态创建Socket时，由操作系统分配一个文件描述符（FD）来代表这个Socket。
2. **建立连接（针对TCP）**：
   - 客户端使用 `connect` 调用与服务器建立连接，而服务器端使用 `accept` 接受客户端的连接。
3. **调用 `recv` 等待数据**：
   - 请求操作系统从Socket的文件描述符对应的缓冲区中读取数据。
   - `recv` 会在没有可用数据时阻塞当前线程，直到有数据到达。这一过程是在内核态进行的，操作系统负责监控网络接口上的数据，并将接收到的数据存储到与该Socket关联的缓冲区中。
4. **数据从内核态传输到用户态**：
   - 一旦有数据可用，操作系统会将数据从内核态的缓冲区复制到用户态应用程序指定的缓冲区中**（只有一次拷贝）**。
5. **处理接收到的数据**：
   - `recv` 调用返回，开始处理这些数据。
6. **关闭Socket**：
   - 数据交换完成后，使用 `close` （在Unix、Linux系统）或 `closesocket` （在Windows系统）关闭Socket。

**Select原理**

从原理可以看出，普通Socket是一条单行道，一个Socket完成，然后才可以进行下一个。而select将单个处理改为了批处理：

1. **准备文件描述符集合**：将所有要监控的Socket文件描述符加入到一个集合中。
2. **拷贝集合到内核**：将这个集合拷贝到内核空间，让操作系统知道需要监控哪些Socket**（一次拷贝）**。
3. **监控描述符集合**：内核遍历这个集合，标记那些有事件（如可读、可写或错误）发生的Socket。
4. **结果拷贝回用户态**：内核将有事件发生的Socket标记集合拷贝回用户空间**（二次拷贝）**。
5. **处理有事件的Socket**：应用程序遍历这个标记集合，对有事件发生的Socket执行相应操作。

由于不再局限于一次只处理一个Socket，那么Select也可以设置返回条件，来进行下一批处理。

1. **超时时间**：当调用 `select` 函数时可以指定一个超时值。
2. **处理就绪Socket的时间**：
   - 当 `select` 返回后，你需要检查哪些Socket就绪，并对这些Socket进行处理（如读取、写入或接受连接）。处理这些Socket所花费的时间也会影响到整个循环的间隔。
   - 这个处理时间取决于你的应用程序逻辑和Socket的数量。

我们补充上Select的代码：

```C++
void AnkiSocket::startSocket(char *command) {
    if (strcmp(command, "server") == 0) {
        std::cout << "Please select socket link mode: " << std::endl;
        std::cout << "1. Single-threaded Mode" << std::endl;
        std::cout << "2. Multi-threaded Mode " << std::endl;
        std::cout << "3. Select Mode " << std::endl;
        int mode;
        std::cin >> mode;
        if (mode != 1 && mode != 2 && mode != 3) {
            std::cout << "Unrecognized Mode";
            return;
        }
        serverSocket();
        if (mode == 1) {
            while(1){
                SOCKET SingleSocket = createClntSocket();
                //可选择业务
                ReceiveFile(SingleSocket);
            }
        }
        if (mode == 2) {
            while (1) {
                SOCKET newClntSocket = createClntSocket();
                std::thread newClntSocketThread(ReceiveFile, newClntSocket);
                newClntSocketThread.detach();
            }
        }
        if (mode == 3) {
            // 初始化套接字集合和变量
            fd_set master_set;
            SOCKET max_sd = linkSocket, newClntSocket;
            FD_ZERO(&master_set); // 清空套接字集合
            FD_SET(linkSocket, &master_set); // 将linkSocket添加到集合中

            while (true) {
                fd_set read_fds = master_set; // 从master_set复制到read_fds，用于select调用

                // 等待套接字活动，无超时设置
                int activity = select(max_sd + 1, &read_fds, NULL, NULL, NULL);
                if ((activity < 0) && (errno != EINTR)) {
                    errorMessage("Select error."); // select调用错误
                    continue;
                }
                std::cout << "select socket ok: " << activity << std::endl;

                // 检查是否有新的连接请求
                if (FD_ISSET(linkSocket, &read_fds)) {
                    newClntSocket = createClntSocket(); // 创建新的客户端套接字
                    if (newClntSocket == -1) {
                        break;
                    }
                    FD_SET(newClntSocket, &master_set); // 将新套接字加入集合
                    max_sd = std::max(newClntSocket, max_sd); // 更新最大套接字描述符
                }

                // 检查和处理所有套接字的数据接收
                for (int i = 0; i <= max_sd; i++) {
                    if (i != linkSocket && FD_ISSET(i, &read_fds)) {
                        ReceiveFile(i); // 接收从套接字i传来的数据
                        FD_CLR(i, &master_set); // 从集合中移除套接字i
                    }
                }
            }
        }
    } else {
        std::cout << "Please select your file. " << std::endl;
        clientSocket();
    }
}
```

这段代码相比上一段添加了mode3，用于实现select功能。

我们使用mode3进行测试：运行发送端`./AnkiSocket.exe client`，选择文件时暂停一下，客户端会有打印回显。代表连接成功了。此时我们再新运行一个发送端，选择文件时，客户端会有一个新的打印。此时先选择新客户的文件发送，再选择旧的，服务端会先接收新客户端的文件，后接收旧的文件，异步成功！

和mode1对比，我们同时打开两个客户端，今后都可以connect成功，然后让你选择发送的文件，但是服务端只会等待第一个客户端先发送，第二个一定要等待第一个结束才可以。通过加入打印，可以发现服务端不会出现第二个socket创建成功的提示。

##### epoll

Todo..

## 示例

本文代码均通过个人项目测试：

[AnkiLink]: https://github.com/HarrySunV9x/AnkiLink/tree/main/exe
