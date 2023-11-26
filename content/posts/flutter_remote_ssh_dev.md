---
title: "Flutter远程开发环境记录"
subtitle: ""
date: 2023-11-26T18:46:26+08:00
lastmod: 2023-11-26T18:46:26+08:00
draft: false
author: "moexco"
authorLink: ""
description: ""

tags: []
categories: []

hiddenFromHomePage: false
hiddenFromSearch: false

featuredImage: ""
featuredImagePreview: ""

toc:
  enable: true
math:
  enable: false
lightgallery: false
license: ""
---

![miku镇楼 pid:113672669](/flutter_remote_ssh_dev/頬杖ミクちゃん_113672669.png)

> ArchLinux环境下的Flutter远程开发环境配置记录文，如有参考请注意时效性

## Flutter

### 安装

- 安卓需要jdk8和jdk17，所以提前安装  
`pacman -Sy jdk8-openjdk jdk17-openjdk`  

- flutter和android-sdk  
`pacman -S flutter android-sdk`  
> `ArchLinux`可以从`archlinuxcn`源里安装`flutter`，开启源的方法我就不再赘述了，直接参考[清华源](https://mirrors.ustc.edu.cn/help/archlinuxcn.html)的使用文档  
>
>
> `android-sdk`是Android开发必要的东西，如果只是远程`flutter web`则不需要  

- 如果启用archlinuxcn之后安装程序报密钥错误,可运行以下两个命令处理  
```sh
pacman-key --init  
pacman-key --populate  
```

### 配置

使用软件源安装的好处是可以跟随系统滚动更新，但是同样带来了一些使用细节上的问题需要处理

#### 1、信任目录  
```sh
git config --global --add safe.directory /opt/flutter

# 如何使用的用户不是root，从源里安装android-sdk时默认路径是`/opt/android-sdk`，后续安装工具链需要授予普通用户权限
chown -R $USER:$USER /opt/android-sdk
```
> `/opt/flutter`是archlinux源安装flutter的默认路径，如果不是在这个路径则需要自己替换真实路径

#### 2、通过`sdkmanager`安装安卓开发相关工具

- 切换jdk版本，因为 **sdkmanager 最高只支持jdk8**  
```sh
# 以下命令都需要以su权限运行
# 查看当前jdk版本
archlinux-java status

# 设置默认jdk版本
archlinux-java set java-8-openjdk
```

- 添加sdk环境变量  
```sh
export PATH=/opt/android-sdk/tools/bin:$PATH
export PATH=/opt/flutter/bin:$PATH
```
- 安装必要工具链  
```sh
# Android开发需要三个东西，都可以用sdkmanager来安装

# 先列一下sdkmanager的基本使用命令
sdkmanager --list # 列出所有可安装或已安装的东西
sdkmanager --install "target_name;version" # 安装指定版本的sdk

# cmdline-tools
sdkmanager --install "cmdline-tools;latest"
# platform-tools
sdkmanager --install "platform-tools"
# platforms
sdkmanager --install "platforms;android-33"
```

- 检查  
安装完成后，可以运行`flutter doctor`检查一下输出  
```sh
[root@archlinux ~]# flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
Could not parse java version from:

If there is a version please look for an existing bug https://github.com/flutter/flutter/issues/ and if one does not
exist file a new issue.
[✓] Flutter (Channel stable, 3.16.0, on Arch Linux 6.5.7-arch1-1, locale C.UTF-8)
[!] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
    ✗ Could not determine java version
    ✗ Android license status unknown.
      Run `flutter doctor --android-licenses` to accept the SDK licenses.
      See https://flutter.dev/docs/get-started/install/linux#android-setup for more details.
[✗] Chrome - develop for the web (Cannot find Chrome executable at google-chrome)
    ! Cannot find Chrome. Try setting CHROME_EXECUTABLE to a Chrome executable.
[✗] Linux toolchain - develop for Linux desktop
    ✗ clang++ is required for Linux development.
      It is likely available from your distribution (e.g.: apt install clang), or can be downloaded from
      https://releases.llvm.org/
    ✗ CMake is required for Linux development.
      It is likely available from your distribution (e.g.: apt install cmake), or can be downloaded from
      https://cmake.org/download/
    ✗ ninja is required for Linux development.
      It is likely available from your distribution (e.g.: apt install ninja-build), or can be downloaded from
      https://github.com/ninja-build/ninja/releases
    ✗ pkg-config is required for Linux development.
      It is likely available from your distribution (e.g.: apt install pkg-config), or can be downloaded from
      https://www.freedesktop.org/wiki/Software/pkg-config/
[!] Android Studio (not installed)
[✓] Connected device (1 available)
[✓] Network resources

! Doctor found issues in 5 categories.
```

可以看到`Android toolchain`这一栏是一个感叹号，问题出在java版本不对和`licenses`，现在处理一下这两个问题

- 切换jdk17并执行licenses
```sh
archlinux-java set java-17-openjdk
flutter doctor --android-licenses
```
再次运行`flutter doctor`检查状态，可以看到前两项都打勾了
```sh
[root@archlinux ~]# flutter doctor
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.16.0, on Arch Linux 6.5.7-arch1-1, locale C.UTF-8)
[✓] Android toolchain - develop for Android devices (Android SDK version 30.0.3)
```

## ssh远程adb

安装好环境，还需要能连上设备，不同于常规的安卓开发，远程开发无法使用现成的虚拟机和真机，需要我们自己转发一下adb的连接

### ssh隧道

重新声明，Flutter开发环境放在远程主机上，去远程的设备如笔记本，则是本机设备

使用过vscode远程开发的人都体验过vscode自带的端口转发和隧道功能，没有vscode的情况下，我们就需要自己用ssh操作了

最简单的方式就是`ssh -L 本地主机端口号:localhost:远程主机端口号 user@ip`  

于是，转发adb的流量命令就是`ssh -L 5037:localhost:5037 user@ip`

> 在通过ssh建立隧道之前，需要先关掉本地设备上的adb服务，否则5037端口号被占用无法打通  
> 关闭命令: `adb kill-server`

### ssh config持久化

每次开发前都运行一下`ssh -L`就太麻烦了，可以直接在ssh的config文件里配置好

在`~/.ssh/config`文件中添加一下格式的内容
```sh
Host studio                             # Host 你自定义的名字
  HostName 192.168.123.123              # HostName 远程主机的IP地址或域名
  Port 22                               # Port 远程主机的ssh端口号，不写默认22
  User moexco                           # User ssh登录的用户名，不写默认root
  LocalForward 5037 localhost:5037      # adb端口隧道的配置
```


### 远程adb连接真机

> 打开手机无线adb，连接跟本地设备同一个局域网络，这个操作略过

ssh转发5037端口之后，在**远程**或**本地**设备，正常使用adb连接设备即可

以下是我连接真机的例子
```sh
> $ adb pair 192.168.123.115:41021
Enter pairing code: 570121
Successfully paired to 192.168.123.115:41021 [guid=adb-99231FFAZ008ES-1ggJAb]

> $ adb connect 192.168.123.115:40827
connected to 192.168.123.115:40827

> $ adb devices
List of devices attached
192.168.123.115:40827   device
```

adb连上设备之后，正常运行flutter项目即可
```sh
flutter run -d 192.168.123.115:40827
```

