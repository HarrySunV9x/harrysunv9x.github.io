---
title: Socket编程
data: 2024-04-02
---

# Socket编程

## 背景

以C/C++语言为例，详细阐述Socket编程过程。

## 过程

**初始化**

第一步是初始化环境。对于Linux环境，直接引入所需头文件即可：

```c++
#include <sys/socket.h>							// 引入socket数据类型与方法，如struct sockaddr、socket()、bind()等
#include <unistd.h>								// 与操作系统交互的方法，如close()
#include <netinet/in.h>							// 包含了用于IP地址和端口号等数据结构，如IPPROTO_IP
```

对于Windwos环境，除了引入头文件，还需要对系统接口初始化：

```c++
#include "winsock.h"									// Windows的socket都在这个头文件里

/* 初始化WindowsAPI */
WSADATA wsaData;										// 初始化变量
if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {		// 传入版本与变量并启动
	std::cout << "WSA Start Up Error" << std::endl;
	return;
}
std::cout << "WSA Start Up" << std::endl;
```

除了程序代码意以外，在编译时还需要挂载系统自带的外部库：

```cmake
...
target_link_libraries(socket_example_code wsock32)		# 以CMake配置文件为例，引入wsock32
```

**错误信息**

为了获取Socket连接中的异常状态，我们要在每一步都添加错误处理，这不仅在Socket中很有用。

在Linux中，实际的错误码存储在全局变量 `errno` 中。我们可以通过库函数strerror(errno)很容易的获取错误信息：

```c++
printf("Linux Socket Have A Error: %s", strerror(errno));
```

而Windows的处理就麻烦很多了，我们需要通过Windows接口得到类似errno一样的错误信息，并对其解析。

我们可以自定义一个方法来显示错误信息：

```C++
// 错误消息打印函数
void errorMessage(std::string errorMessage) {
    int wsaLastError = WSAGetLastError();       //用于在调用 Winsock 函数出错时获取错误代码
    if (wsaLastError == 0) {
        std::cerr << "Unidentified Problem, please check the remote." << std::endl;
        return;
    }

    std::vector<char> errorText(256);
    // 调用 FormatMessageA 来获取错误消息
    FormatMessageA(
            FORMAT_MESSAGE_FROM_SYSTEM, // 使用系统消息表来格式化错误代码
            nullptr,                    // 没有指定额外的消息源
            wsaLastError,               // 要格式化的错误代码，通常是 WSAGetLastError 的返回值
            0,                          // 选择合适的语言ID，0 表示使用默认语言
            errorText.data(),           // 指向存储错误消息的缓冲区的指针
            errorText.size(),           // 缓冲区的大小
            nullptr);                   // 没有使用可变参数列表
    std::cerr << errorMessage << wsaLastError << " - " << errorText.data() << std::endl;
}

errorMessage("Windows Socket Have A Error: ");
```

**对象创建**

在Linux的编程艺术里，有一个哲学思想：

> 一切皆文件(Everything is a file)

无论是数据、驱动、设备、接口，在Linux里的任何东西都是是用文件表示了。本质上，就是有要有这么一个对象，让我们可以操作它。

```C++
int socketFD;					// 在Linux中声明变量
SOCKET win_socket;				// 在Windows中声明变量
```

不同的操作系统、不同的语言创建socket对象的方式有所不同。Linux中，socketFD就是一个int类型数据，而Windows中与define好的unsigned int数据。

Linux与Windows调用接口的方式基本相同，但返回值略有不同，比如Linux创建socket失败是返回-1，而Windows是返回0，因此对于错误处理要有一些区别。

对于Socket的赋值：

```C++
socketFD = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
if (socketFD < 0) {
	printf("Linux Create Socket Failed. err: %s", strerror(errno));		// 错误打印
	return -1;															// 自定义错误处理
}

win_socket = socket(AF_INET, SOCK_STREAM, IPPROTO_IP);
if (win_socket == INVALID_SOCKET) {
    errorMessage("Windows Create Socket Failed. err: ");				// 错误打印
    return -1;															// 自定义错误处理
}
```

二者都可以通过socket赋值，且传递的参数也已知，只是返回值不同。

AF_INET：地址族（Address Family），AF_INET表示因特网地址族，如UDP, TCP等。

SOCK_STREAM：Socket类型，这里使用流式传输。还有数据报传输、原生套接字等等。

IPPROTO_TCP：Socket使用的协议，这里定义了TCP协议。

对于TCP传输，用上述定义即可，其他常用的设置：

```c++
socket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);						// UDP 套接字（无连接的不可靠数据传输）：
socket = socket(AF_INET, SOCK_RAW, IPPROTO_RAW);						// 原始套接字（用于直接访问网络协议）：
socket = socket(AF_INET6, SOCK_STREAM, IPPROTO_TCP);					// 使用 IPv6 的 TCP 套接字：
socket = socket(AF_INET6, SOCK_DGRAM, IPPROTO_UDP);						// 使用 IPv6 的 UDP 套接字：

socket = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);						// 原始 ICMP 套接字
socket = socket(AF_INET, SOCK_RAW, IPPROTO_IGMP);						// 原始 IGMP 套接字
socket = socket(AF_INET, SOCK_RAW, IPPROTO_ARP);						// 原始 ARP 套接字
socket = socket(AF_INET, SOCK_RAW, IPPROTO_IP);							// 原始 IP 套接字

socket = socket(AF_PACKET, SOCK_RAW, htons(ETH_P_ALL));					// 串口套接字（用于串口通信）：
socket = socket(AF_BLUETOOTH, SOCK_STREAM, BTPROTO_RFCOMM);				// 蓝牙套接字（用于蓝牙设备之间的通信）：
```

要注意的是，尽管保证兼容性，Windows和Linux的宏定义命名大致一致，但在使用时，还需要检查一下是否有对应的接口支持。

**域绑定**

域名、协议、端口，这三者的组合为域。

当做服务端，等待外来的Socket连接时，Socket首先要做一个域绑定。

```C++
struct sockaddr_in socket_addr;													// 声明地址参数
memset(&sockAddr, 0, sizeof(sockAddr));											// 初始化socket_addr为0

socket_addr.sin_family = AF_INET;												// 协议
socket_addr.sin_addr.s_addr = inet_addr("127.0.0.1");							// 域名
socket_addr.sin_port = htons(10086);											// 端口

if (bind(socketFD, (struct sockaddr *)&socket_addr, sizeof(address)) < 0) {		
	printf("Linux Bind Socket Failed. err: %s", strerror(errno));				// 错误打印
	return -1;																	// 自定义错误处理
}
```

Linux与Windows可以使用相同的代码进行绑定。这里的bind第二个参数是struct sockaddr *，sockaddr 是比 sockaddr_in 更通用的数据结构。

**监听**

服务端绑定好域后，则可以开始监听了，有两步骤：

①建立监听服务器

```c++
if (listen(socketFD, 10) < 0) {											// 第二个参数代表支持的连接数
	printf("Listen Socket Failed. err: %s", strerror(errno));			// 错误打印
	close(socketFD);													// 监听失败关闭Socket
	return -1;															// 自定义错误处理
}
```

②接收服务端连接

```c++
int accept_socket = accept(socketFD, NULL, NULL);								// Linux
SOCKET accept_socket = accept(linkSocket, NULL, NULL);							// Windows

if (accept_socket < 0) {														// Linux
if (accept_socket == INVALID_SOCKET) {											// Windows
    printf("Accept Socket Failed. err: %s", strerror(errno));
    return -1;
}
```

accept的第二个和第三个参数会给传入的参数设置sockaddr和socklen，如果有需要，可以传入两个变量的地址赋值。

这里和创建Socket一样，二者的返回值不同。

**连接**

服务端开启监听后，客户端就可以连接了。

```c++
if (connect(socketFD, (struct sockaddr *) &socket_addr, sizeof(socket_addr)) < 0) {
    printf("Connect Socket Failed. err: %s", strerror(errno));
    return -1;
}
```

客户端的connect和服务端的bind参数是一致的。本质上，东西是一样的，一个是我的，一个是我连你的。

**收发**

客户端与服务端建立好连接后，就可以进行收发了，

```c++
char sendBuffer[1024];
char message[11] = "I Love You";
sprintf(sendBuffer, "I want tell you: %s", message);									// 格式化字符串

if (send(socketFD, sendBuffer, strlen(sendBuffer), 0) != strlen(sendBuffer)) {
    printf("Send Socket Failed. err: %s", strerror(errno));								// 错误打印
    close(socketFD);																	// 发送失败关闭Socket
    return -1;
}
```

本文使用了sprintf来格式化发送的字符串。如果使用了C++的`std::string`，可以调用`.c_str()`来格式成char数组。

Linux的send返回的是发送的长度，而Windows则表示是否发送成功.

如果是发送的字符串，要用strlen，足够就好。如果是指针数据，则用sizeof，保证数据发送完整。

对于接收来说，考虑的要比发送更多一些，先看下最基本的接收数据：

```c++
char recvBuff[1024];
int readLen = recv(socketFd, recvBuff, 1024, 0);
if(readLen < 0) {
    printf("Recv Socket Failed. err: %s", strerror(errno));							// 错误打印
    close(socketFD);																// 接收失败关闭Socket
    return -1;
}
```

基于网络情况，在接收时候我们往往容易碰到粘包的情况，Socket很可能会一次收到两个包。或者丢包，只有半个包，一个半的包等情况。为了提高健壮性，我们最好对发送的数据添加一个结束标志位。

```c++
char sendBuffer[1024];
char message[11] = "I Love You";
sprintf(sendBuffer, "I want tell you: %s;end;", message);	
```

这样，当我们碰到`end;`的时候，就知道一个数据接收完成了。我们新增一个变量用作大缓存，每次recv的数据放在小缓存。然后每一次读取数据处理完整的帧。

```C++
int FindEnd(char *receiveBuff) {
	char *end;
	if (receiveBuff == NULL) {
		return 0;						// 外传指针检空
	}
	
	end = strstr(receiveBuff, "end;");	// 找到结束标志位的位置
	if (end != NULL) {
		*end = '\0';					// 设置结束标志位，字符串的长度检测就只到这里了
		return 1;
	}
	return 0;
}

char recv[1024];
int recvLen = 0;

int recvFunction() {
	char recvOnce[256];
	int recvOnceLen;
	
	while(1) {
		recvOnceLen = recv(socketFd, recvOnce, 256, 0);
        if(recvOnceLen < 0) {
            printf("Recv Socket Failed. err: %s", strerroor(errno));	
            close(socketFD);
            return -1;
        }
        
        if (memcpy_s(&recv[recvLen], 1024, recvOnce, recvOnceLen) != 0) {
        	recvLen = 0;
        	printf("Recv Overflow");	
        	return -1;
        }
        
        recvLen += recvOnceLen;
        recv[recvLen] = '\0';
        
        while(FindEnd(recv)) {
            int moveLen = 0;
            moveLen = strlen(recv) + 4;			// 4为end;的长度，这里moveLen指向下一个包的起点
            ...									// 处理recv
            recvLen -= moveLen;
            
            memmove(recv, &recv[moveLen], recvLen + 1);
            // 第一个参数：要复制的位置
			// 第二个参数：要复制的数据的起始位置。
			// 第三个参数，要复制的字节的数量
        }
	}
}
```

memmove的理解如图：

![image-20240403103620754](.\memmove图示.png)