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

以传输文件（TCP）为例，介绍Socket的实现。

### Windows端

#### 服务端

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

#### 客户端

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



### Android（Linux）端

接收端代码在大部分方面保持一致，唯一不同的是接口不同。在这里，只涉及到服务端的接收功能。我们不再分段，直接提供完整的代码段：

```C++
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Starting AnkiSocket");
//创建套接字
serv_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
if (serv_sock == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Socket creation failed: %s",
                        strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Socket created successfully");
// 将套接字和IP、端口绑定
memset(&serv_addr, 0, sizeof(serv_addr));  // 清空地址结构
serv_addr.sin_family = AF_INET;            // 使用IPv4地址
serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");  // 设置IP地址
serv_addr.sin_port = htons(50000);         // 设置端口号
    // 绑定套接字
if (bind(serv_sock, (struct sockaddr *) &serv_addr, sizeof(serv_addr)) == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Bind failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Bind successful");
// 监听套接字，设置最大等待连接数
if (listen(serv_sock, 20) == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Listen failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Listening");
// 接受客户端连接请求
struct sockaddr_in clnt_addr;
socklen_t clnt_addr_size = sizeof(clnt_addr);
int clnt_sock = accept(serv_sock, (struct sockaddr *) &clnt_addr, &clnt_addr_size);
if (clnt_sock == -1) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Accept failed: %s", strerror(errno));
    return;
}
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Client connected");
// 接收文件名
char fileNameBuffer[260];
int fileNameLength = recv(clnt_sock, fileNameBuffer, 260, 0);
if (fileNameLength <= 0) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Failed to receive file name: %s",
                        strerror(errno));
    close(clnt_sock);
    return;
}
std::string receivedFileName = fileNameBuffer;
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Received file name");
// 构造文件的完整路径
std::string internalPath = "/data/data/com.harry.apk/";
std::string fullPath = internalPath + receivedFileName;
std::ofstream outfile(fullPath, std::ofstream::binary);
// 检查文件是否成功打开
if (!outfile) {
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Cannot open file: %s",
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
    __android_log_print(ANDROID_LOG_ERROR, "AnkiLink", "Error receiving data: %s",
                        strerror(errno));
    outfile.close();
    close(clnt_sock);
    return;
}
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "File transfer completed");
// 关闭文件
outfile.close();
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "File closed");
// 发送接收确认消息
const char *RECEIPT_CONFIRMATION = "RECEIPT_CONFIRMED";
send(clnt_sock, RECEIPT_CONFIRMATION, strlen(RECEIPT_CONFIRMATION) + 1, 0);
// 关闭客户端套接字
close(clnt_sock);
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Client socket closed");
// 关闭套接字
close(serv_sock);
__android_log_print(ANDROID_LOG_INFO, "AnkiLink", "Socket closed");
```

