---
title: Flutter环境搭建
data: 2024-03-07
---

# Flutter环境搭建

即便官网有了中国区的教程，但是说明还是不够很详细，踩了很多坑，这边简单介绍一下Flutter的搭建过程。

### 1：设置环境变量

为了确保Flutter能够在您的计算机上顺利运行，我们需要先设置一些环境变量。这些变量将帮助Flutter识别必要的资源和路径。

1. `PUB_HOSTED_URL`:  https://pub.flutter-io.cn
2.  `FLUTTER_STORAGE_BASE_URL`:https://storage.flutter-io.cn
3. `NO_PROXY`: `localhost,127.0.0.1,::1`
4. `ALL_PROXY`（如果需要）: `http://代理地址:端口` 或 `socks5://代理地址:端口`。

前面两个为中国区镜像，后面两个是代理配置。

### 2：添加PATH(windows)/Install(Linux\Mac)

-  `C:\Flutter\bin`（假设将Flutter安装在C:\Flutter）。
-  `C:\AndroidSDK`（假设将AndroidSDK安装在C:\AndroidSDK）。

### 3：下载与安装Flutter SDK

1. [下载Flutter SDK](https://flutter.cn/docs/release/archive?tab=windows)。
2. 解压下载的文件到 `C:\Flutter` 目录（注意：路径中不要包含空格）。

### 4：安装Android Studio和Visual Studio 2022

安装Android Studio和Visual Studio 2022，这些IDE为Flutter开发提供了必要的工具和支持。

1. 从Android Studio官网下载并安装Android Studio。
2. 在安装时选择安装Android SDK（推荐版本34）。
3. 从[Visual Studio官网](https://visualstudio.microsoft.com/)下载并安装Visual Studio 2022。
4. 在安装Visual Studio时，确保包含C++开发环境。

### 5：设置Android cmdline-tools

1. 下载Android cmdline-tools并解压到AndroidSDK目录。
2. 在cmdline-tools的bin目录下执行以下命令：
   - `.\sdkmanager.bat --install "cmdline-tools;latest"`
   - `flutter doctor --android-licenses`

### 6：更换Maven镜像地址：

1. 打开`/path-to-flutter-sdk/packages/flutter_tools/lib/src/http_host_validator.dart`文件，修改`https://maven.google.com/`为 google maven 的国内镜像，如`https://maven.aliyun.com/repository/google/`
2. 删除`/path-to-flutter-sdk/bin/cache` 文件夹

### 7：运行flutter doctor

最后，运行 `flutter doctor` 命令检查环境是否已正确配置，如果有问题会有对应回显。