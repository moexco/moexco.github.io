---
title: "Linux触屏旋转"
subtitle: ""
date: 2021-03-26T23:03:02+08:00
lastmod: 2021-03-26T23:03:02+08:00
draft: false
author: "moexco"
authorLink: ""
description: ""

tags: ["Linux"]
categories: ["Linux"]

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

![miku镇楼 pid:51832998](/linux-touch-screen-rotation/无题_51832998.png)

## 说明

其实这是一个回忆记录贴，前几天朋友提到了这件事，所以想记录一下

> 起因是这样的：<br /><br />
> 我收了一个x86的板子，酷比IWork10 Pro 旗舰版，具体型号没记错是I15-TC，原想是用来看视频(~~顺便推推gal~~)，但是事与愿违，这个板子带win10着实性能捉急，而寨板嘛。。。。驱动原因又装不了win8，那只能考虑一下Linux了 <br /> <br />
> Linux的驱动倒是基本都有，至少我用到的都有了，但是吧，就是有一个严重的问题，触屏反了。。。。 <br /> <br />
> 当时因为种种原因，我没有选择重新编译(~~实在是没有编译设备啊，也不会有人想用Windows编译东西吧~~)，而是修改一下打个补丁凑合用用




## 安装xinput和libinput

这里需要使用xinput查看一下触控信息，所以要安装xinput和libinput

安装没有什么难度，简单提供一下Debian系列和Arch系列的安装命令：<br />
Debian/Ubuntu: `sudo apt install xserver-xorg-input-libinput` <br />
Arch/Manjaro: `sudo pacman -S xf86-input-libinput`

如果是Debian/Ubuntu的话，使用`sudo apt install xinput`命令安装，如果是其他的请自行查看包管理安装

> 当然，还是推荐查看各自系统的安装文档

## 查看触控信息

在终端键入命令`xinput --list`

它应该会打出大量的信息，如果你的打印是
```
bash: xinput: command not found
```

请返回上一步，安装`xinput`

正确的信息应该长这样： <br />
```
moexco@ArchLinux:~ $ xinput --list
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ Logitech USB Optical Mouse                id=6    [slave  pointer  (2)]
⎜   ↳ WaveShare WaveShare Touchscreen           id=7    [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    ↳ Logitech USB Keyboard                     id=8    [slave  keyboard (3)]
    ↳ Logitech USB Keyboard                     id=9    [slave  keyboard (3)]
```

随后我们要在这里面找到`Touchscreen`字样的一项，比方说我上面的信息里的`WaveShare WaveShare Touchscreen           id=7    [slave  pointer  (2)]`

现在我们要确认一下这个是不是走了`libinput`，键入命令: `xinput --list-props 7`，这里的`7`是上面的`id=7`

应该会打印出类似如下的信息：<br />
```
moexco@ArchLinux:~ $ xinput --list-props 7
Device 'WaveShare WaveShare Touchscreen':
        Device Enabled (115):   1
        Coordinate Transformation Matrix (116): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        Device Accel Profile (240):     0
        Device Accel Constant Deceleration (241):       1.000000
        Device Accel Adaptive Deceleration (242):       1.000000
        Device Accel Velocity Scaling (243):    10.000000
        Device Product ID (244):        3823, 5
        Device Node (245):      "/dev/input/event3"
        Evdev Axis Inversion (246):     0, 0
        Evdev Axis Calibration (247):   <no items>
        Evdev Axes Swap (248):  0
        Axis Labels (249):      "Abs MT Position X" (267), "Abs MT Position Y" (268), "Abs MT Pressure" (269), "None" (0), "None" (0), "None" (0)
        Button Labels (250):    "Button Unknown" (233), "Button Unknown" (233), "Button Unknown" (233), "Button Wheel Up" (121), "Button Wheel Down" (122)
        Evdev Scrolling Distance (251): 0, 0, 0
        Evdev Middle Button Emulation (252):    0
        Evdev Middle Button Timeout (253):      50
        Evdev Third Button Emulation (254):     0
        Evdev Third Button Emulation Timeout (255):     1000
        Evdev Third Button Emulation Button (256):      3
        Evdev Third Button Emulation Threshold (257):   20
        Evdev Wheel Emulation (258):    0
        Evdev Wheel Emulation Axes (259):       0, 0, 4, 5
        Evdev Wheel Emulation Inertia (260):    10
        Evdev Wheel Emulation Timeout (261):    200
        Evdev Wheel Emulation Button (262):     4
        Evdev Drag Lock Buttons (263):  0
```
OK，看到`Coordinate Transformation Matrix (116): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000`，这就是我们要的信息了，这说明是`libinput`方案

当然，还有另一种是`Evdev`，也就是上面的：<br />
```
        Evdev Scrolling Distance (251): 0, 0, 0
        Evdev Middle Button Emulation (252):    0
        Evdev Middle Button Timeout (253):      50
        Evdev Third Button Emulation (254):     0
        Evdev Third Button Emulation Timeout (255):     1000
        Evdev Third Button Emulation Button (256):      3
        Evdev Third Button Emulation Threshold (257):   20
        Evdev Wheel Emulation (258):    0
        Evdev Wheel Emulation Axes (259):       0, 0, 4, 5
        Evdev Wheel Emulation Inertia (260):    10
        Evdev Wheel Emulation Timeout (261):    200
        Evdev Wheel Emulation Button (262):     4
        Evdev Drag Lock Buttons (263):  0
```
如果有看到类似这些的话，可以使用使用`Evdev`


## libinput

### 根据需要旋转矩阵

从上面我们可以看到，屏幕的触控其实就是那一串数字：<br />
```
1 0 0
0 1 0
0 0 1
```
在正常情况下，上面的样子就是正确的，但是我们现在要修改一下触控的位置，也许是旋转，也许是交换x轴，也许是交换y轴，而我的酷比平板遇到的情况是，点击屏幕左上角，实际出发点击的是在屏幕右上角，也就是说，我需要交换x轴坐标

这里提供一些几种常见的结果：<br />

* 只交换x轴<br />
  `-1 0 1 0 1 0 0 0 1`
* 只交换y轴<br />
  `1 0 0 0 -1 1 0 0 1`
* 交换x轴和y轴(即镜像翻转)<br />
  `-1 0 1 0 -1 1 0 0 1`
* 90度旋转<br />
  `0 1 0 -1 0 1 0 0 1`
* 180度旋转<br />
  `-1 0 1 0 -1 1 0 0 1`
* 270度旋转<br />
  `0 -1 1 1 0 0 0 0 1`


### xorg修改方法

> Linux Desktop 的古老方案就是X11和xorg，当然xorg的所有输入都是经由X11所以，从X11的配置下手，改一改应该就好了



#### 临时修改(测试效果)

选定了修改后的触控矩阵之后，可以先在终端试一下，用如下命令：<br />
`xinput set-prop 'WaveShare WaveShare Touchscreen' 'Coordinate Transformation Matrix' -1 0 1 0 1 0 0 0 1`

`WaveShare WaveShare Touchscreen`是驱动名，根据上面打印出来的修改，`Coordinate Transformation Matrix`是我们要修改的，所以是固定的，而最后的就是矩阵参数了

理论上运行完之后就生效了，你可以用这个方式测试哪个矩阵参数适合你的情况

#### 持久化修改

每次都运行一下那个命令着实有些不优雅，其实xinput是可以写入到配置文件里的

首先复制一下`/usr/share/X11/xorg.conf.d/40-libinput.conf`到`/etc/X11/xorg.conf.d/`下，如果etc下的目录不存在就手动创建，如果conf文件名字数字不一样没关系，复制就行了

然后修改`/etc/X11/xorg.conf.d/40-libinput.conf`文件：<br />

找到`touchscreen`项: <br />
```
 Section "InputClass"
        Identifier "libinput touchscreen catchall"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
 EndSection
```

在里面添加一行，变成这样：<br />
```
 Section "InputClass"
        Identifier "libinput touchscreen catchall"
        Option "CalibrationMatrix" "-1 0 1 0 1 0 0 0 1"
        MatchIsTouchscreen "on"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"
 EndSection
```

重启就生效了

### wayland修改方法

Wayland并不能像xorg一样，修改配置文件，xinput命令也修改不了，根据arch wiki的说明，只能靠系统提供修改接口才行。

可我就是要改

我们还有一个必胜法：`/etc/udev/rules.d/libinput.rules`文件

选定矩阵参数之后，可以通过如下命令修改：<br />
`echo 'ENV{LIBINPUT_CALIBRATION_MATRIX}="-1 0 1 0 1 0 0 0 1"' > /etc/udev/rules.d/libinput.rules`


## Evdev 

> 鉴于我并没有使用该方案，所以这里简单说一下，具体请自行查阅文档说其他说明

### xorg

#### 临时修改(测试效果)

跟libinput的类似，xorg可以通过xinput命令先测试，这里提供几个常见的：<br />
* 正常<br />
  ```
  xinput set-prop 7 "Evdev Axis Inversion" 0, 0
  xinput set-prop 7 "Evdev Axes Swap" 0
  ```
* 左旋转<br />
  ```
  xinput set-prop 7 "Evdev Axis Inversion" 1, 0
  xinput set-prop 7 "Evdev Axes Swap" 1
  ```
* 颠倒<br />
  ```
  xinput set-prop 7 "Evdev Axis Inversion" 1, 1
  xinput set-prop 7 "Evdev Axes Swap" 0
  ```
* 右旋转<br />
  ```
  xinput set-prop 7 "Evdev Axis Inversion" 0, 1
  xinput set-prop 7 "Evdev Axes Swap" 1
  ```

> 简单解释一下<br />
> `xinput set-prop 7 "Evdev Axes Swap" 1`的`7`是上面的`id=7`，`1`是“交换x轴和y轴”，如果是`0`就是不交换<br />
> `xinput set-prop 7 "Evdev Axis Inversion" 0, 1`的`0, 1`，第一个数字代表x轴，第二个数字代表y轴

#### 持久化修改

跟libinput一样，也可以持久化，Evdev的配置文件，内容应该是这样的：<br />
```
Section "InputClass"
        Identifier	"calibration"
        MatchProduct	"SYNA2393:00 06CB:19AC"
        Option	"Calibration"	"2 3021 -7 2015"
        Option	"SwapAxes"	"0"
EndSection
```

跟测试命令一样，修改里面的相应项就好了，因为我没有使用这个方案，所以比较粗糙，具体的请自行测试了