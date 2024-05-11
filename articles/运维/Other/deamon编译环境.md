# 安装WSL

```
wsl --install
```

> # SSH 公钥设置
>
> 来源于：https://help.gitee.com/base/account/SSH%E5%85%AC%E9%92%A5%E8%AE%BE%E7%BD%AE
>
> Gitee 提供了基于 SSH 协议的 Git 服务，在使用 SSH 协议访问仓库仓库之前，需要先配置好账户 SSH 公钥。
>
> > 仓库公钥（部署公钥）请移步 [添加部署公钥](https://help.gitee.com/repository/ssh-key/generate-and-add-ssh-public-key)
>
> ## 生成 SSH 公钥
>
> > Windows 用户建议使用 **Windows PowerShell** 或者 **Git Bash**，在 **命令提示符** 下无 `cat` 和 `ls` 命令。
>
> 1、通过命令 `ssh-keygen` 生成 SSH Key：
>
> ```bash
> ssh-keygen -t ed25519 -C "Gitee SSH Key"
> ```
>
> - `-t` key 类型
> - `-C` 注释
>
> 输出，如：
>
> ```bash
> Generating public/private ed25519 key pair.
> Enter file in which to save the key (/home/git/.ssh/id_ed25519):
> Enter passphrase (empty for no passphrase):
> Enter same passphrase again:
> Your identification has been saved in /home/git/.ssh/id_ed25519
> Your public key has been saved in /home/git/.ssh/id_ed25519.pub
> The key fingerprint is:
> SHA256:ohDd0OK5WG2dx4gST/j35HjvlJlGHvihyY+Msl6IC8I Gitee SSH Key
> The key's randomart image is:
> +--[ED25519 256]--+
> |    .o           |
> |   .+oo          |
> |  ...O.o +       |
> |   .= * = +.     |
> |  .o +..S*. +    |
> |. ...o o..+* *   |
> |.E. o . ..+.O    |
> | . . ... o =.    |
> |    ..oo. o.o    |
> +----[SHA256]-----+
> ```
>
> - 中间通过三次**回车键**确定
>
> 2、查看生成的 SSH 公钥和私钥：
>
> ```bash
> ls ~/.ssh/
> ```
>
> 输出：
>
> ```bash
> id_ed25519  id_ed25519.pub
> ```
>
> - 私钥文件 `id_ed25519`
> - 公钥文件 `id_ed25519.pub`
>
> 3、读取公钥文件 `~/.ssh/id_ed25519.pub`：
>
> ```bash
> cat ~/.ssh/id_ed25519.pub
> ```
>
> 输出，如：
>
> ```bash
> ssh-ed25519 AAAA***5B Gitee SSH Key
> ```
>
> 复制终端输出的公钥。
>
> ## 设置账户 SSH 公钥
>
> 用户可以通过主页右上角 **「个人设置」->「安全设置」->「SSH 公钥」->「[添加公钥](https://gitee.com/profile/sshkeys)」** ，添加生成的 public key 添加到当前账户中。
>
> > 需要注意： **添加公钥需要验证用户密码**
>
> ![添加账户 SSH 公钥](https://help.gitee.com/assets/images/sshkeys_create-8409f453e6780ca1a8db3ce33c74240b.png)
>
> 通过 `ssh -T` 测试，输出 SSH Key 绑定的**用户名**：
>
> ```bash
> $ ssh -T git@gitee.com
> Hi USERNAME! You've successfully authenticated, but GITEE.COM does not provide shell access.
> ```
>
> 在添加完公钥后，用户可以在 **「个人设置」->「安全设置」->「[SSH 公钥](https://gitee.com/profile/sshkeys)」** 浏览查看当前账户已经添加的 SSH 公钥，并对公钥进行管理/删除操作。
>
> ![浏览 SSH Key](https://help.gitee.com/assets/images/sshkeys_list-bff1a324894abbdc3ab8f61c49bb63d5.png)
>
> ![查看/删除 SSH Key](https://help.gitee.com/assets/images/sshkeys_show-a14cdfb89475debed237bfded2bd9848.png)
>
> ## 仓库的 SSH Key 和账户 SSH Key 的区别？
>
> 账户的 SSH Key 和账户绑定，当账户具有 **推送/拉取** 权限时可通过 SSH 方式 **推送/拉取** 的仓库。
>
> 通过 `ssh -T` 测试时，输出 SSH Key 绑定的用户名：
>
> ```bash
> $ ssh -T git@gitee.com
> Hi USERNAME! You've successfully authenticated, but GITEE.COM does not provide shell access.
> ```
>
> 仓库的 SSH key 只针对仓库，且我们仅对仓库提供了部署公钥，即仓库下的公钥仅能**拉取**仓库，这通常用于生产服务器拉取仓库的代码。
>
> 通过 `ssh -T` 测试时，输出 Anonymous：
>
> ```bash
> ssh -T git@gitee.com
> Hi Anonymous! You've successfully authenticated, but GITEE.COM does not provide shell access.
> ```
>
# 配置公钥

1. 进入环境

   ```
   wsl --distribution Ubuntu20.04
   ```

2. 设置公钥

   ```
   ssh-keygen -t ed25519 -C "Gitee SSH Key"
   ```

3. 添加到gitee

4. 获取权限

   ```
   sudo su
   ```

5. 创建工作目录

   ```
   mkdir /home/deamon
   cd /home/deamon
   ```

6. 配置应用仓

   ```
   vim /etc/apt/sources.list
   ```

   ```
   deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
   deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
   deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
   ```

   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```

7. 安装git:

   ```
   sudo apt install git-all	
   ```

8. 安装git-lfs

   ```
   curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh
   sudo apt-get install git-lfs
   git lfs install
   ```

9. 安装repo工具

   ```
   curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > /usr/local/bin/repo
   chmod a+x /usr/local/bin/repo
   
   apt install python3-pip
   pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple requests
   ```

10. 下载主分支代码

    ```
    git config --global user.email "harrysunv9x@outlook.com"
    git config --global user.name "HarrySunV9x"
    
    sudo ln -s /usr/bin/python3 /usr/bin/python
    repo init -u git@gitee.com:openharmony/manifest.git -b master --no-repo-verify
    repo init -u https://gitee.com/openharmony/manifest --no-repo-verify
    repo sync -c
    repo forall -c 'git lfs pull'
    ```

11. 安装依赖

    ```
    sudo apt-get update && sudo apt-get install binutils binutils-dev git git-lfs gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib gcc-arm-linux-gnueabi libc6-dev-i386 libc6-dev-amd64 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip m4 bc gnutls-bin python3.8 python3-pip ruby genext2fs device-tree-compiler make libffi-dev e2fsprogs pkg-config perl openssl libssl-dev libelf-dev libdwarf-dev u-boot-tools mtd-utils cpio doxygen liblz4-tool openjdk-8-jre gcc g++ texinfo dosfstools mtools default-jre default-jdk libncurses5 apt-utils wget scons python3.8-distutils tar rsync git-core libxml2-dev lib32z-dev grsync xxd libglib2.0-dev libpixman-1-dev kmod jfsutils reiserfsprogs xfsprogs squashfs-tools pcmciautils quota ppp libtinfo-dev libtinfo5 libncurses5-dev libncursesw5 libstdc++6 gcc-arm-none-eabi vim ssh locales libxinerama-dev libxcursor-dev libxrandr-dev libxi-dev
    ```

12. 

    ```
    
    ```

