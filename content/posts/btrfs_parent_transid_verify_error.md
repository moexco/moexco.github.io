---
title: "btrfs parent transid verify error的玄学修复记录"
subtitle: ""
date: 2023-10-15T16:29:37+08:00
lastmod: 2023-10-15T16:29:37+08:00
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

## 错误排查

> 小伙伴大半夜找我说暗影大人的视频无法播放了，我去确认了一下，jellyfin转码烧录mks字幕确实出问题了，但是让客户端不转码直接本地播放是可以播放的，就没在意，直到第二天一觉醒来，他又来说问题了，这次不是视频无法播放，而是jellyfin打不开了
>
> 我第一反应是ddns寄了，或者我路由网络又出毛病了(~~昨晚才处理好的Clash DNS解析啊~~)，但是一个测试发现并没有，只是所有web服务都无法访问了，ssh看了下docker，好家伙docker寄了
>
> 好端端的docker怎么会寄，考虑到docker容器所在的分区曾经寄过，我dmesg看了下，嗯，磁盘分区寄了，症状和上次类似，只是这一次寄得更彻底了。

首先，寄了的问题磁盘分区是ArchLinux上的`sdb1`：
```sh
Disk /dev/sdb: 1.75 TiB, 1920383410176 bytes, 3750748848 sectors
Disk model: SDLF1CRR-019T-1H
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: B9A58F50-EE49-49CF-825C-A96C869F0C67

Device          Start        End    Sectors  Size Type
/dev/sdb1        2048 2702172847 2702170800  1.3T Linux filesystem
/dev/sdb2  2702174208 3750748159 1048573952  500G Linux filesystem
```

而这是`dmesg`记录的一部分日志，~~因为这是事后回忆整理，我留下的记录并不是很多，悲~~
```sh
[204772.045179] BTRFS warning (device sdb1): checksum verify failed on logical 114737152 mirror 1 wanted 0xa1e021a8 found 0x0944aebf level 0
[204772.045592] BTRFS warning (device sdb1): checksum verify failed on logical 114737152 mirror 2 wanted 0xa1e021a8 found 0x0944aebf level 0
[204772.045632] BTRFS: error (device sdb1: state A) in __btrfs_update_delayed_inode:1065: errno=-5 IO failure
[204772.045662] BTRFS info (device sdb1: state EA): forced readonly
[204772.045670] BTRFS: error (device sdb1: state EA) in __btrfs_run_delayed_items:1156: errno=-5 IO failure
[204772.045690] BTRFS warning (device sdb1: state EA): Skipping commit of aborted transaction.
[204772.045696] BTRFS: error (device sdb1: state EA) in cleanup_transaction:1997: errno=-5 IO failure
[210074.918333] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 1 wanted 1 found 0
[210074.918670] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 2 wanted 1 found 0
[210074.919073] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 1 wanted 1 found 0
[210074.919330] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 2 wanted 1 found 0
[210074.919657] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 1 wanted 1 found 0
[210074.920055] BTRFS error (device sdb1: state EA): level verify failed on logical 771833856 mirror 2 wanted 1 found 0
```

## 尝试找到错误文件

可以看到错误首先是`114737152`这个inode对应的文件校验错误，那么首先应该找到它，好在分区虽然寄了，但是因为`forced readonly`所以依旧能挂载并且可读，先一个`find ./ -num 114737152`尝试找到它，结果并没有像以前寄的时候那样找到对应的文件，而是直接打印了一大堆的报错
```sh
cannot access filename: Input/output error
```

## 通过btrfs工具查看错误

btrfs虽然是以不成熟和不稳定且性能一般闻名于世，不过还是提供了一些工具来尝试解决常见问题

>通过谷歌搜索，我找到了一个类似问题的[疑难解答页面](https://btrfs.readthedocs.io/en/latest/trouble-index.html#error-parent-transid-verify-error)，本来以为是人家单独列出来肯定是常见好救的，结果它建议我备份数据，这是一个永久性错误，并且原因不明

**这不能够啊！**

~~这个文档里虽然没有提供任何修复方案，但是记录了btrfs的各种命令用法，倒也不算没有一点用处~~

### 确认一下btrfs记录到多少个错误

`btrfs dev stats /dev/sdb1`跑一下，~~上一次分区错误的时候，这里记录了几百个错误，但是没有处理好，我最后格式化分区了~~，出乎意料的，这次一个错误都没有发现
```sh
> $ sudo btrfs dev stats /dev/sdb1
[/dev/sdb1].write_io_errs    0
[/dev/sdb1].read_io_errs     0
[/dev/sdb1].flush_io_errs    0
[/dev/sdb1].corruption_errs  0
[/dev/sdb1].generation_errs  0
```

虽然不知道这意味着什么，但没有错误记录，也许还有救，在网上找一找前人的经验，比方说[StackOverflow上的帖子](https://stackoverflow.com/questions/46472439/fix-btrfs-btrfs-parent-transid-verify-failed-on)

### 尝试`usebackuproot`

>按照帖子里的一个答案，只读挂载时带上`usebackuproot`：  
>`mount -t btrfs -o ro,usebackuproot /dev/sdb1 /home/cache`  
>如果能够成功挂载，则去掉只读的`ro`重新挂载，能挂载上说明修复了

我尝试只读挂载成功并没有问题，~~毕竟我本来就能只读挂载~~，此时再次进入`/home/cache`目录尝试读取文件，依旧是`Input/output error`，抱着侥幸心理，我尝试不指定只读挂载，即`mount -t btrfs -o usebackuproot /dev/sdb1 /home/cache`，不出意料的发生了错误，~~但是错误信息我没保留下来，所以没法展示了~~

### 格式化前的死马当活马医

> 前文说过，我这个分区不是第一次出现这种错误，上一次我尝试了各种btrfs的命令也没修复成功，但成功把数据给备份出来了
> 而这一次我讲数据复制出来都会发生错误，其实并没有什么动力去研究，毕竟有一次失败在前，而且这次连备份都做不到，甚至产生放弃btrfs，格式化回ext4的想法

就在我即将运行`mkfs.ext4 /dev/sdb1`时，我突然想到这次`btrfs dev stats`并没有报错，虽然出现了上次没有出现的`Input/output error`，但是btrfs本身的情况也许没那么糟糕，上次用过的命令可以试试

`btrfs`有不少的子命令工具可以做一些诸如重建树或者擦除

先跑一个`btrfs scrub status /home/cache`看看结果
```sh
> $ sudo btrfs scrub status /home/cache
UUID:             74cd7002-8aaf-4421-b4f4-4320ee12850c
        no stats available
Total to scrub:   256.06GiB
Rate:             0.00B/s
Error summary:    no errors found
```

**没有错误**

> btrfs有一个不建议使用的命令可以尝试修复错误，即`btrfs check --repair /dev/sdb1`，我上一次也执行过，但并没有修复我的问题

**但这次既然没有错误，也就是有尝试的价值**

直接运行`btrfs check --repair --force /dev/sdb1`，果然跟上一次什么变化都没有的输出不同，它输出了很多修复节点的信息，~~也没有保存下来~~

运行完成之后，我尝试可读写挂载，问题依旧

### 峰回路转

失望也带着绝望，我只能格式化了，并且这次格式化我不打算再使用btrfs了，它已经两次在同一个分区上寄了，`mkfs.ext4 /dev/sdb1`，跟数据和btrfs说拜拜吧 ~~其实我挂载的路径是cache就说明了没有什么重要数据，仅仅是缓存分区，随便格的那种~~

> 出乎意料的，格式化没有开始，设备繁忙中

我确认了一下，我已经卸载设备了，并没有挂载到目录，`btrfs dev stats`也显示设备被占用

也许是check占用了设备没有释放，此时的我也没有心情去关心了，只想快点格了然后重建docker容器恢复服务

关闭docker服务开机自启，关闭systemd的开机自动磁盘挂载，重启设备一气呵成

> 开机之后，我正要格式化，又想，也许它好了呢？

于是我只读挂载了一下，正常挂载，甚至执行`ls`的时候都没有`Input/output error`了，并且`dmesg`也没有任何错误信息，我心情有点小激动，二话不说`systemctl start docker`，看看docker能起来不，起不来，依旧是报错，报错信息我都懒得看了

**等会，我现在是只读挂载，docker怎么可能起来！！**

**改成可读写挂载，docker启动了！！服务上线了！！**

### 后续

访问一下jellyfin，发现页面有了，但登录页打不开，`F12`看一下原因，少了一个js文件，直接重建容器，一切都正常了，也许`btrfs check --repair`给我移除了无法通过校验的文件

**准确说我不清楚我是怎么修复的，我只能靠回忆记录下这个并没有多少操作的过程，其中我如果有那么一次没有想再看看分区是不是好了，或者check之后设备不是繁忙中，我恐怕不敢相信我居然能看到它恢复正常，这大概就是不放弃吧(bushi**

<!--more-->
