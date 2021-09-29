---
title: "Aio服务器搭建"
subtitle: ""
date: 2021-09-22T14:48:13+08:00
lastmod: 2021-09-22T14:48:13+08:00
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


# 个人Aio服务器搭建

**以下是碎碎念，建议跳过**

> 百度云限速、360云盘迅雷云盘等相继关闭；QQ音乐、网易云音乐版权竞争，歌单里的歌三天两头灰色，独占的独占，曲库缺失的缺失，为了听歌还要每个平台都开会员，并且体验依旧割裂；爱奇艺异军突起、腾讯视频独霸江湖、优酷抱紧了老剧，哔哩哔哩看个纪晓岚标注着1080P事实上并不达标；广电要求影视作品一律先审后播，b站在抽帧黑幕圣光之后直接连追番都不行了，没法同步上映......与此同时，网上一大堆家庭NAS方案层出不穷......

所以我不禁开始思考，已经在上班的我，是时候回收一下中学的梦想————想要一台自己的服务器。

常年的网易云会员，b站会员现在也都已经过期了，促使我不再续费的原因，不外乎网易云越来越多的灰色，哔哩哔哩越来越少且不一定能同步更新的番，老实说，会员不贵，但体验实在是太割裂了。只能去特定平台才能看到特定的东西，对于视频其实还好，切一个app，但是音乐咧，我只想随机我所有的歌，我没法把几个平台的歌放在一起随机。

总之，作为正版流媒体平台的受害者，我选择离开流媒体平台，支持正版的方式有非常的多，比方说买专辑，买光碟，但是，我需要一个允许我在任何地方在线播放的平台。

所以，我需要一个NAS。

但显然不够，作为一个coding玩家，我还需要一个开发环境，我需要一个运行环境等等等等

所以，All In One，好巧不巧，我家有公网IP，而我又有域名，Cloudflare又可以白嫖DDNS，走起~

## 平台和硬件选择

### 低功耗 or 高性能

确定了要AIO，那自然需要考虑一个严重的问题，传统NAS上肯定是追求低功耗的，并且不要求多高的性能，树莓派都可以胜任，甚至还有路由器刷op挂一个samba直接用的。

当然了，NAS其实还有很多现成的方案，群晖啊，威联通啊什么的，而硬件上，也有之前的矿机什么星际蜗牛啊，最早的J1900到后来的J3455，可以说是五花八门了。

不过都有一个共同特点，恰好都不是我现在入手能捞到实惠的东西，车那是一去不复返，鱼塘全涨价了。

还有一个很重要的点，我不打算花很多钱，我预算1000以内弄完除硬盘以外的东西，然后花500左右整硬盘。

并且我需要足够高的性能，J1900这些是不太行了，J3455绝对就超预算了。

最后选择了闻名江湖的E5神教！

### 配置单

```
主板：华南X79豪华板
CPU：E5 2650L V2
网卡：Intel 82576
电源：500W银欣(型号忘了)
散热：酷冷至尊(未知型号)
显卡：黄伟达 GTX 1050(真实型号估计是GTS 450)
内存：16*2 + 8*2 总计48G的四通道DDR3 ECC
固态：影驰 铁甲战将 240G SATA3
机械：HGST 矿盘 3T*2
```
除掉硬盘，算上邮费，总共花了750，邮费就84了,
两块HGST，赶上了涨价，花了454,
所以总共1204，还可以吧，预算内

这一套下来满载上200W了，不过考虑到大部分时间是基本上空载的，当然算不上省电，根据网友的测试，空载不到20W，一个月十几块钱电费吧(-_-

### 系统和软件

一般人就是直接黑群晖了，但咱是一般人嘛，咱是折腾佬啊

想都没想就是`手搓KVM虚拟化`，从此踩坑无数后续章节慢慢补坑

# 正片

## 一、Arch物理系统安装
   
首先贴上Arch的官方wiki的[中文安装文档](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

~~好了，本章完结~~

**以下注意事项**
1. fdisk工具不支持gpt格式，也不支持2T以上容量的分区操作，所以我建议用`parted`代替，当然这个工具也是安装在Arch的livecd里了
2. `/etc/locale.gen`文件一定要编辑，然后用`locale-gen`生成信息，并且在`/etc/locale.conf`文件中至少写上`LANG=en_US.UTF-8`，否则将无法在终端显示中文，即使你安装了中文字体
3. 在重启到安装好的系统之前，网络工具一定要装，这里推荐一下`networkmanager`，安装命令`sudo pacman -Sy networkmanager`，然后设置开机启动`sudo systemctl enable NetworkManager`
4. 安装好系统之后，文档里有简单的提了一下安装引导的事情，如果你不想开不了机，就记得点一下[`GRUB`](https://wiki.archlinux.org/title/GRUB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))进去看一下关于引导的安装
5. 包管理器我建议使用`yay`而不是`pacman`，它们的命令是基本上一样的，但`yay`支持从`archlinuxcn`和`AUR`下载软件包，这显然比`pacman`更方便，以下的命令我将使用`yay`而不是`pacman`

> 因为AIO服务器的重点并不在物理机的系统安装上，而且网上有太多的同类教程可供参考，所以我就略过了，只标注了我认为需要注意一下的东西


## 二、KVM虚拟化

安装好物理机之后，接下来就是虚拟化，显然现在有非常多的懒人解决方案，集成度非常的高，效果也不错，有基于Debian的各种系统，也有不基于任何平台的单纯WEB管理的各种虚拟化管理页面，but，本着不对外提供服务的东西不需要可视化界面的原则，以及对KVM虚拟化的好奇，我不使用各种WEB或GUI管理工具。

同样，我会在开头，将ArchWiki的文档[放在这里](https://wiki.archlinux.org/title/KVM_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，使用Arch的好处就是相当完善的文档，并且中文支持不错，当然我还是建议尽可能的查看英文文档，毕竟更新及时.

### 检查虚拟化支持

纯软件虚拟化不是不行但是效率实在实在太低了，所以现在都是从硬件平台上就开始支持各种虚拟化技术，也就是说，你需要先在主板的BIOS上打开虚拟化的开关，然后在系统里确认对虚拟化的支持状态。

1. 在终端执行`LC_ALL=C lscpu | grep Virtualization`或`grep -E --color=auto 'vmx|svm|0xc0f' /proc/cpuinfo`，输出什么东西并不是太重要，重要的是不能没有输出，如果没有任何东西输出的话，你就需要去主板检查一下bios是不是没有正确打开虚拟化的开关
   
    ![LC_ALL=C lscpu | grep Virtualization](/aio-server/kvm-lscpu.png)
2. 在确定了硬件支持之后，就需要确定软件是否安装了，执行`zgrep CONFIG_KVM /proc/config.gz`查看Linux内核的虚拟化模块是否安装，如果输出的结果不是`y`或者`m`，则说明虚拟化模块没有安装到位。~~因为我没有遇到这个问题，所以我只能将安装模块的文档[放在这里](https://wiki.archlinux.org/title/Kernel_module_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))，如果你遇到了，请自行解决~~
   
    ![zgrep CONFIG_KVM /proc/config.gz](/aio-server/kvm-config-gz.png)
3. 之后需要确定内核模块是否已经加载，执行`lsmod | grep kvm`，如果执行了之后没有输出东西，但是上一步又确定了已经安装模块，那就需要[手动加载](https://wiki.archlinux.org/title/Kernel_modules_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%89%8B%E5%8A%A8%E5%8A%A0%E8%BD%BD%E5%8D%B8%E8%BD%BD)。~~同样的，我也没有遇到这个问题~~
   
    ![lsmod | grep kvm](/aio-server/kvm-lsmod.png)

到这里就差不多了，当然还有其他的东西可以使用，比方说`VIRTIO`模块、嵌套虚拟化之类的东西，可以参考上面的文档自行检查。
    

### 安装软件包

在Arch安装软件一直都是一件轻松愉快的事情，archlinuxcn也好AUR也好，可谓是独步天下

不过我这里需要说明的事情是，KVM是一种虚拟化技术，而不是一个虚拟机，我们需要一个虚拟机来应用这种技术，在Linux平台上，那自然是`QEMU`了

同样，[文档](https://wiki.archlinux.org/title/QEMU_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))就放在这了

因为我不启用`GUI`，也就是不安装桌面，所以我不需要对图形化的支持，在安装`qemu`的时候我选择`qemu-headless`，如果你有图形化的需求，则安装`qemu`

安装命令`yay -Sy qemu-headless`

#### `QEMU`简单跑个win虚拟机

> 运行Windows虚拟机跟直接跑Linux虚拟机不太一样，所以我单独列了一个小节，如果你不打算运行Windows虚拟机，那可以跳过，注意，本小节旨在将Windows虚拟机运行并完成安装，不会涉及到显卡直通或虚拟化显卡的内容

##### 准备文件

* 因为微软❤️开源，所以我们没法直接在qemu虚拟机上启动Windows的镜像，我们需要一个辅助镜像virtio-win，Arch可以直接通过包管理下载到，执行命令`yay -Sy virtio-win`，然后这个镜像会被下载到`/var/lib/libvirt/images/virtio-win.iso`，将它复制到跟我们Windows系统镜像一个目录下就行了。

* 下载Windows的系统镜像，比方说我下载的镜像重命名为`win10_2004.iso`。~~主要是原来的名字太长了~~

* 现在启动镜像和系统镜像都有了，我需要创建一个虚拟磁盘，就像装物理机的系统需要一块硬盘一样，虚拟机一样需要能识别到一个磁盘空间，执行命令`qemu-img create -f qcow2 win10.qcow2 100G`，这里我创建了一个叫`win10.qcow2`的镜像文件，镜像格式是`qcow2`，容量大小是100G，不过不用担心你的物理空间没那么大，它不会直接占用100G的空间，是动态占用的，不过你依然至少保留20G的空闲空间。

至此，我们必要的文件就已经准备好了

##### 启动win10虚拟机

`qemu`跟VMWare和VBox那些不一样，它不具备管理和持久化虚拟机的能力，你需要手动配置每一个参数，然后以命令的形式运行，运行之后不会保存参数，你下次运行依旧需要输入那些参数，所以我们创建一个`shell`文件来运行，省得敲那么长的命令

并且我们不直接使用`qemu`命令，因为qemu软件包已经提供了一个不同架构的命令，比方说我们跑64位x86的虚拟机就用`qemu-system-x86_64`。~~用这个已经很繁琐了，就不用更繁琐的`qemu`了~~

执行`vim win.sh`然后复制下面的东西
``` s
qemu-system-x86_64 \
        -enable-kvm \
        -m 8G \
        -smp 20 \
        -display default \
        -vga virtio \
        -machine q35,vmport=off,dump-guest-core=off,mem-merge=off,hmat=on,usb=off \
        -drive file=win10.qcow2,index=0,media=disk,if=virtio \
        -drive file=virtio-win.iso,index=1,media=cdrom \
        -drive file=win10_2004.iso,index=2,media=cdrom \
        -spice port=5900,addr=0.0.0.0,disable-ticketing=on
```
> 别忘了把文件名和路径换成你们自己的，还有一些参数，比方说`-smp 20`是20核，建议改成你的电脑的真实配置，包括`-m 8G`是虚拟机分配的8G内存

然后给它执行权限`chmod +x win.sh`，再运行它就可以启动了，但是因为我们没有图形化，看不到启动，这就需要我们从其他电脑远程了，上面的脚本里我使用了`spice`来做远程，自行从互联网下载一个支持连接`spice`的客户端连接即可。

剩下的事情就是正常的安装和重启，就可以看到一个**非常卡**的win10虚拟机在跑了

### Libvirt 管理KVM

`qemu`非常的强但是不好用，所以我们需要一个能像正常虚拟机一样管理它的，这里就不得不提`libvirt`，它提供了一些C实现的后端接口，和一个能用的前端`virsh`

#### 安装软件包

安装依旧很简单，只需要执行`yay -Sy libvirt`，但是只有这个是不够的，不同于上面小节中简单的启动一个虚拟机，我们需要管理一些东西，比方说网络，所以我们还需要一些其他的东西

执行 `yay -Sy libvirt bridge-utils dnsmasq`

#### 启动服务

`libvirt`是一个常驻服务，所以需要启动，执行`sudo systemctl start libvirtd`，不过这样每次都需要手动启动，执行`sudo systemctl enable libvirtd`实现开机自启

同时，我们需要启动virsh自带的NAT网络，执行`sudo virsh net-start default`

可以执行`sudo virsh net-list --all`查看网络是否启动了，如果有一个`defalut`网络那就是启动了

##### 使用原生virsh启动虚拟机

创建`win10.xml`文件，复制下面的内容，修改之后，执行`virsh define win10.xml`可以增加一个虚拟机，并且受`virsh`管理。
``` xml
<domain type="kvm">
  <name>win10</name>
  <memory unit="GiB">8</memory>
  <vcpu placement="static">20</vcpu>
  <os>
    <type arch="x86_64" machine="pc-i440fx-xenial">hvm</type>
  </os>
  <cpu mode="host-passthrough">
    <topology sockets="2" cores="10" threads="2"></topology>
  </cpu>
  <devices>
    <disk type="file" device="disk">
      <driver name="qemu" type="qcow2" cache="default"></driver>
      <source file="win10.qcow2"></source>
    </disk>
    <disk type="file" device="cdrom">
      <source file="win10_2004.iso" index="2"></source>
    </disk>
    <disk type="file" device="cdrom">
      <source file="virtio-win.iso" index="1"></source>
    </disk>
    <graphics type="spice" port="5900" listen="0.0.0.0" passwd="yourpassword"></graphics>
  </devices>
</domain>
```

> 虽然这是核心玩法，但是手写xml文件真的太麻烦了，而且参数实在太多了，所以还是需要一个东西，能帮我们手动生成一些模板配置，只需要在这个基础上简单修改我们的参数，这就是`virt-install`了。

#### 安装`virt-install`

一样直接安装即可`yay -Sy virt-install`

> 安装这东西让我纠结了非常久，因为我本来是不想在物理机上装任何执行环境，哪怕是Python，但是手写xml真的太麻烦了......这之间我发现了有另一个`libvirt-go`的项目，我在想如果有一个`virt-install-go`的项目就好了，这样不需要依赖也能直接用，不过目前是没有的，虽然可以自己实现，我自己稍微写了一下，这个周期实在太长了，而且`libvirt-go`本身也还有问题，文档也非常的缺，只能先用`virt-install`这个py的版本了......

#### 使用`virt-install`创建win10虚拟机

执行
``` s
virt-install \
--name Win10 \
--memory 8192 \
--vcpus sockets=1,cores=10,threads=2 \
--os-type=windows \
--os-variant=auto \
--cdrom=win10_2004.iso \
--disk virtio-win.iso,device=cdrom,bus=ide \
--disk win10.qcow2,bus=virtio,size=100,format=qcow2 \
--graphics spice,listen=::,password=yourpassword \
--hvm \
--autostart \
--virt-type kvm
```

然后就可以用`spice`连接过去看了，本地的窗口运行可以`Ctrl+C`关闭了

#### `virsh`管理虚拟机

**注意: virsh命令不需要root权限**

* 查看虚拟机列表：     `virsh list --all`
* 关闭虚拟机(不删除)： `virsh destroy name`
* 删除虚拟机(不关闭):  `virsh undefine name`
* 设置虚拟机开机自启： `virsh autostart name`
* 取消虚拟机开机自启： `virsh autostart --disable name`

> 如果你的虚拟机正在运行，没有先执行关闭就执行了删除，虚拟机不会被马上删除，会等到关闭之后才删除

## 三、openwrt旁路由
## 四、虚拟机映射物理分区
## 五、docker容器化服务管理
## 六、samba文件服务配置
## 七、jellyfin家庭影音搭建

<!--more-->
