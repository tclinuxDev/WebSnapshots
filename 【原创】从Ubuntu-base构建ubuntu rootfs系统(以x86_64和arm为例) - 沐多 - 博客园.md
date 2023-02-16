---
created: 2023-02-16T13:29:20 (UTC +08:00)
tags: []
source: https://www.cnblogs.com/wsg1100/p/13127636.html
author: 沐多
            粉丝 - 83
            关注 - 7
        
    
    
    
    
                +加关注
---

# 【原创】从Ubuntu-base构建ubuntu rootfs系统(以x86_64和arm为例) - 沐多 - 博客园

> ## Excerpt
> 版权声明：本文为本文为博主原创文章，转载请注明出处，博客地址：https://www.cnblogs.com/wsg1100/。如有错误，欢迎指正。 1.介绍 ubuntu-base 是Ubuntu官

---
版权声明：本文为本文为博主原创文章，转载请注明出处，博客地址：[https://www.cnblogs.com/wsg1100/。如有错误，欢迎指正。](https://www.cnblogs.com/wsg1100/%E3%80%82%E5%A6%82%E6%9C%89%E9%94%99%E8%AF%AF%EF%BC%8C%E6%AC%A2%E8%BF%8E%E6%8C%87%E6%AD%A3%E3%80%82)  

-   -   [1.介绍](https://www.cnblogs.com/wsg1100/p/13127636.html#1%E4%BB%8B%E7%BB%8D)
    -   [2.目的](https://www.cnblogs.com/wsg1100/p/13127636.html#2%E7%9B%AE%E7%9A%84)
    -   [2.准备宿主系统](https://www.cnblogs.com/wsg1100/p/13127636.html#2%E5%87%86%E5%A4%87%E5%AE%BF%E4%B8%BB%E7%B3%BB%E7%BB%9F)
        -   [2.1 宿主系统需求](https://www.cnblogs.com/wsg1100/p/13127636.html#21-%E5%AE%BF%E4%B8%BB%E7%B3%BB%E7%BB%9F%E9%9C%80%E6%B1%82)
            -   [附:创建回环文件并分区](https://www.cnblogs.com/wsg1100/p/13127636.html#%E9%99%84%E5%88%9B%E5%BB%BA%E5%9B%9E%E7%8E%AF%E6%96%87%E4%BB%B6%E5%B9%B6%E5%88%86%E5%8C%BA)
                -   [(1). 创建镜像文件](https://www.cnblogs.com/wsg1100/p/13127636.html#1-%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F%E6%96%87%E4%BB%B6)
                -   [(2). 建立分区](https://www.cnblogs.com/wsg1100/p/13127636.html#2-%E5%BB%BA%E7%AB%8B%E5%88%86%E5%8C%BA)
                    -   [a.分区方式一](https://www.cnblogs.com/wsg1100/p/13127636.html#a%E5%88%86%E5%8C%BA%E6%96%B9%E5%BC%8F%E4%B8%80)
                    -   [b.分区方式二](https://www.cnblogs.com/wsg1100/p/13127636.html#b%E5%88%86%E5%8C%BA%E6%96%B9%E5%BC%8F%E4%BA%8C)
                -   [(3). 映射分区设备并格式化](https://www.cnblogs.com/wsg1100/p/13127636.html#3-%E6%98%A0%E5%B0%84%E5%88%86%E5%8C%BA%E8%AE%BE%E5%A4%87%E5%B9%B6%E6%A0%BC%E5%BC%8F%E5%8C%96)
                -   [(4). 挂载分区](https://www.cnblogs.com/wsg1100/p/13127636.html#4-%E6%8C%82%E8%BD%BD%E5%88%86%E5%8C%BA)
        -   [2.2 挂载新分区](https://www.cnblogs.com/wsg1100/p/13127636.html#22-%E6%8C%82%E8%BD%BD%E6%96%B0%E5%88%86%E5%8C%BA)
        -   [2.3 配置目标Ubuntu源](https://www.cnblogs.com/wsg1100/p/13127636.html#23-%E9%85%8D%E7%BD%AE%E7%9B%AE%E6%A0%87ubuntu%E6%BA%90)
        -   [2.3 配置DNS](https://www.cnblogs.com/wsg1100/p/13127636.html#23-%E9%85%8D%E7%BD%AEdns)
        -   [2.3 配置用户创建默认配置文件](https://www.cnblogs.com/wsg1100/p/13127636.html#23-%E9%85%8D%E7%BD%AE%E7%94%A8%E6%88%B7%E5%88%9B%E5%BB%BA%E9%BB%98%E8%AE%A4%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
        -   [2.3 进入chroot环境](https://www.cnblogs.com/wsg1100/p/13127636.html#23-%E8%BF%9B%E5%85%A5chroot%E7%8E%AF%E5%A2%83)
    -   [3.安装软件](https://www.cnblogs.com/wsg1100/p/13127636.html#3%E5%AE%89%E8%A3%85%E8%BD%AF%E4%BB%B6)
        -   [3.1 基本配置](https://www.cnblogs.com/wsg1100/p/13127636.html#31-%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE)
        -   [3.2安装内核](https://www.cnblogs.com/wsg1100/p/13127636.html#32%E5%AE%89%E8%A3%85%E5%86%85%E6%A0%B8)
        -   [3.3 安装X-window](https://www.cnblogs.com/wsg1100/p/13127636.html#33-%E5%AE%89%E8%A3%85x-window)
    -   [4.安装各种桌面(可选)](https://www.cnblogs.com/wsg1100/p/13127636.html#4%E5%AE%89%E8%A3%85%E5%90%84%E7%A7%8D%E6%A1%8C%E9%9D%A2%E5%8F%AF%E9%80%89)
        -   [4.1 安装Lubuntu的定制LXDE](https://www.cnblogs.com/wsg1100/p/13127636.html#41-%E5%AE%89%E8%A3%85lubuntu%E7%9A%84%E5%AE%9A%E5%88%B6lxde)
        -   [4.2 安装gnome桌面](https://www.cnblogs.com/wsg1100/p/13127636.html#42-%E5%AE%89%E8%A3%85gnome%E6%A1%8C%E9%9D%A2)
        -   [4.3 安装xfce桌面环境](https://www.cnblogs.com/wsg1100/p/13127636.html#43-%E5%AE%89%E8%A3%85xfce%E6%A1%8C%E9%9D%A2%E7%8E%AF%E5%A2%83)
        -   [4.4 安装mate桌面](https://www.cnblogs.com/wsg1100/p/13127636.html#44-%E5%AE%89%E8%A3%85mate%E6%A1%8C%E9%9D%A2)
    -   [5.arm构建简要](https://www.cnblogs.com/wsg1100/p/13127636.html#5arm%E6%9E%84%E5%BB%BA%E7%AE%80%E8%A6%81)
        -   [5.1下载ubuntu-base](https://www.cnblogs.com/wsg1100/p/13127636.html#51%E4%B8%8B%E8%BD%BDubuntu-base)
        -   [5.3 安装qemu](https://www.cnblogs.com/wsg1100/p/13127636.html#53-%E5%AE%89%E8%A3%85qemu)
        -   [5.4 将ubuntu-base解压](https://www.cnblogs.com/wsg1100/p/13127636.html#54-%E5%B0%86ubuntu-base%E8%A7%A3%E5%8E%8B)
        -   [5.4 chroot操作](https://www.cnblogs.com/wsg1100/p/13127636.html#54-chroot%E6%93%8D%E4%BD%9C)
        -   [5.5 最后](https://www.cnblogs.com/wsg1100/p/13127636.html#55-%E6%9C%80%E5%90%8E)
-   [附录](https://www.cnblogs.com/wsg1100/p/13127636.html#%E9%99%84%E5%BD%95)
    -   [arch-chroot源码](https://www.cnblogs.com/wsg1100/p/13127636.html#arch-chroot%E6%BA%90%E7%A0%81)

## 1.介绍

ubuntu-base 是Ubuntu官方构建的ubuntu最小文件系统，包含debain软件包管理器，基础包大小通常只有几十兆，其背后有整个ubuntu软件源支持，ubuntu软件一般稳定性比较好，基于ubuntu-base按需安装Linux软件，深度可定制......，常用于嵌入式rootfs构建。

嵌入式常见的几种文件系统构建方法：busybox、yocto、builroot，我觉得它们都不如Ubuntu方便，强大的包管系统，有强大的社区支持，可以直接apt-get install来安装新软件包。本文介绍了如何基于Ubuntu-base构建完整的ubuntu 系统。对于安装过archlinux或者构建过[LFS](http://www.linuxfromscratch.org/)的朋友，对该方式再熟悉不过了。

ubuntu支持很多架构，arm、X86、powerpc、ppc等，本文主要基于X86\_64为例，arm、arm64也给出了简单的制作步骤，其他架构操作类似。

## 2.目的

从ubuntu最小文件系统ubuntu-base开始，构建一个具有X-windows的完整系统。

## 2.准备宿主系统

### 2.1 宿主系统需求

-   宿主系统是一台linux系统电脑，可以是虚拟机，为避免Linux不同发行版间的差异影响系统构建，推荐使用ubuntu，需要能够连接Ubuntu源（有网络），构建时软件需从源下载安装。另外宿主系统硬件架构与目标系统一致（本文认为两者属同一架构，如果不一致需要使用使用qemu等模拟目标架构）。
-   ubuntu-base包，[下载连接](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu-base/releases/16.04.6/release/)，里面包含arm、X86等架构的Ubuntu所有版本的base包，base包是ubuntu的最小文件系统，大小约几十M。这里使用的是X86-64架构，Ubuntu16.04最新base包：`ubuntu-base-16.04.6-base-amd64.tar.gz`
-   一个磁盘分区≥5GB(也可创建一个环回设备，分区后进行操作，方法自寻搜索），目标系统将构建到该分区上，**根据目的自行预估所需磁盘大小，本文构建一个仅运行QT的文件系统，5GB足够了。**由于直接下载安装，所以对宿主系统存储大小无要求。;  
    本文构建一个X86-64架构，基于Ubuntu 16.04的自定义系统， 本文磁盘分区情况如下：

```
/dev/sda1    #UEFI分区 500MB
/dev/sda2    #宿主系统根分区 20GB
/dev/sda3    #用来构建ubuntu-base 5GB
```

#### 附:创建回环文件并分区

环回（loopback）文件系统是Linux类系统中非常有趣的部分。我们通常是在设备上（例如磁盘分区）创建文件系统。这些存储设备能够以设备文件的形式来使用，比如 /dev/device\_name。为了使用存储设备上的文件系统，我们需要将其挂载到一些被称为挂载点（mount point）的目录上。环回文件系统是指那些在**文件中而非物理设备**中创建的文件系统。我们可以将这些文件作为文件系统挂载到挂载点上。这实际上可以让我们在物理磁盘上的文件中创建逻辑磁盘。

我们的目的：创建一个回环文件模拟一个大小10GB的物理磁盘，并在该回环镜像中分两个区：

-   UEFI分区，大小500M，FAT32格式。
-   根分区（/），大小剩余大小，EXT4格式。

##### (1). 创建镜像文件

也就是创建一个大小10G的文件。

```
$dd if=/dev/zero of=ubuntu_base.img bs=1G count=10
记录了10+0 的读入
记录了10+0 的写出
10737418240 bytes (11 GB, 10 GiB) copied, 84.1916 s, 128 MB/s
```

你会发现创建好的文件大小超过了1GB。这是因为硬盘作为块设备，其分配存储空间时是按照块大小的整数倍来进行的。此时可以直接对这个文件作为**一整个分区**格式化并使用，操作如下。

用`mkfs`命令将1GB的文件格式化成ext4文件系统：

```
$mkfs.ext4 ubuntu_base.img
```

可使用下面的命令就可看到已经是文件系统了：

```
$ file ubuntu_base.img
loobackfile.img: Linux rev 1.0 ext4 filesystem data, UUID=3be1775c-8976-445d-9134-8daabb2bade7 (extents) (64bit) (large files) (huge files)
```

现在就可以挂载环回文件了：

```
$sudo  mkdir /mnt/loopback
$sudo mount -o loop ubuntu_base.img /mnt/loopback
```

`-o loop`用来挂载环回文件系统。

这实际上是一种快捷的挂载方法，我们无需手动连接任何设备。但是在内部，这个环回文件连接到了一个名为`/dev/loop1`或`loop2`的设备上。

我们也可以手动来操作：

```
$sudo losetup /dev/loop1 ubuntu_base.img
$sudo mount /dev/loop1 /mnt/loopback
```

使用下面的方法进行卸载（umount）：

```
$sudo umount /mnt/loopback
```

也可以用设备文件的路径作为umount命令的参数：

```
$sudo umount /dev/loop1
```

以上是将整个文件作为一个分区使用的方法，但我们的目的是在该文件内分多个区，继续。

##### (2). 建立分区

###### a.分区方式一

建立分区可以使用`fdisk`或`parted`工具，本人比较喜欢用`parted`，对于设置启动分区非常方便。

```
$ sudo parted ubuntu_base.img
GNU Parted 3.2
使用 /home/work/loobackfile.img
欢迎使用 GNU Parted! 输入 'help'可获得命令列表.
(parted) 
```

新建UEFI分区：

对于UEFI启动方式，首先要设置创建label为msdos；

```
(parted) mklabel msdos
```

创建分区大小500M：

```
(parted)mkpart primary fat32 0 500MB
```

查看新创建的分区,该分区编号为1：

```
(parted) p
Model:  (file)
磁盘 /home/work/ubuntu_base.img: 10.7GB
Sector size (logical/physical): 512B/512B
分区表：msdos
Disk Flags:

数字  开始：  End    大小   类型     文件系统  标志
 1    512B    500MB  500MB  primary  fat32     lba

(parted)
```

将该分区设置为boot(UEFI启动)分区：

```
(parted) set 1 boot on
(parted) p
Model:  (file)
磁盘 /home/work/ubuntu_base.img: 10.7GB
Sector size (logical/physical): 512B/512B
分区表：msdos
Disk Flags:

数字  开始：  End    大小   类型     文件系统  标志
 1    512B    500MB  500MB  primary  fat32     启动, lba

(parted)
```

设置后通过`p`命令可以看到，标志处多了`启动(boot)`标志。

将剩余空间创建根分区：

```
(parted) mkpart primary ext4  500MB 100%
```

到此两个分区创建完毕，使用`p`查看，`q`保存退出：

```
(parted) p
Model:  (file)
磁盘 /home/work/ubuntu_base.img: 10.7GB
Sector size (logical/physical): 512B/512B
分区表：msdos
Disk Flags:

数字  开始：  End     大小    类型     文件系统  标志
 1    512B    500MB   500MB   primary  fat32     启动, lba
 2    500MB   10.7GB  10.2GB  primary  ext4      lba
```

###### b.分区方式二

使用parted分区，需要自己计算大小，对齐等，难免有些麻烦，可以使用sfdisk来简化分区，得到上述同样的分区效果只需执行：

```
sudo sfdisk --force /dev/sdc << EOF
1,500M,0x0c,*
500M,,0x83
EOF
```

sfdisk会从标准输入读取分区描述信息；每一行描述一个分区，常用格式为：<起始柱面>,<柱面数 量>,<分区ID>,< bootable >。如果参数没有指定，则使用默认值；而<起始柱面>的默认值为当前最小可用的柱面编号。

-   `1,500M,0x0c,*`表示的是，从1扇区开始，分配大小为500M的一个分区，分区类型为`0x0c`表示fat32，'\*'标识这是一个启动分区。
-   `500M,,0x83` 表示从500M大小开始，分配大小为自动分配，即剩余所有大小作为一个分区，`0x83`表示这是一个ext分区。

##### (3). 映射分区设备并格式化

镜像文件已经被分为两个区，但是还没有格式化，要对内部的两个分区进行格式化就需要先将两分区挂载到设备。

有一个更快的方法可以挂载镜像中的所有分区——`kpartx`。它并没有安装默认在系统中，你得使用软件包管理器进行安装：

```
$sudo apt-get install kpartx
```

执行以下命令自动挂载镜像文件中的分区：

```
$sudo kpartx -v -a ubuntu_base.img
add map loop0p1 (253:0): 0 976562 linear 7:0 1
add map loop0p2 (253:1): 0 19994624 linear 7:0 976896
```

这条命令在磁盘镜像中的分区与`/dev/mapper`中的设备之间建立了映射，随后便可以格式化/挂载这些设备。

格式化两个分区：

```
$ sudo mkfs.fat -F 32 /dev/mapper/loop0p1  #loop0p1 回环设备的分区1
$ sudo mkfs.ext4 /dev/mapper/loop0p2#loop0p1 回环设备的分区2
```

##### (4). 挂载分区

格式换完成后就可以挂载两个分区：

```
$ sudo mkdir /mnt/loopback
$ sudo mount /dev/mapper/loop0p2  /mnt/  #挂载根分区
$ sudo mkdir -p /mnt/loopback/boot/efi
$ sudo mount /dev/mapper/loop0p1  /mnt/boot/efi  #挂载uefi启动分区
```

### 2.2 挂载新分区

将目标分区格式化后挂载到`/mnt`目录,如果挂载的是其他目录后面的所有命令均需更改：

```
sudo mkfs.ext4 /dev/sda3
sudo mount /dev/sda3 /mnt
```

将`ubuntu-base-16.04.6-base-amd64.tar.gz`解压到`/mnt`目录：

```
sudo tar -xpvf ubuntu-base-16.04.6-base-amd64.tar.gz -C /mnt
```

注意：需要保留ubuntu-base中的文件权限及所有者，解压时需要root权限或者sudo操作，且使用`-p`参数保留权限。

### 2.3 配置目标Ubuntu源

ubuntu-base中有默认ubuntu官方源，如果连接上互联网且访问官方源速度不受限制的话不需要更换。配置文件`/etc/apt/sources.list`更换源,**本文宿主系统与要构建的系统硬件架构及版本一致，所以直接拷贝即可**。

```
sudo cp /etc/apt/sources.list /mnt/etc/apt/
```

### 2.3 配置DNS

进入目标环境需要联网，需要先配置DNS，拷贝宿主系统文件`/etc/resolv.conf`到`/mnt/etc/`目录下：

```
sudo cp /etc/resolv.conf /mnt/etc/
```

### 2.3 配置用户创建默认配置文件

ubuntu-base默认只有root用户，如果需要像普通ubuntu那样可随意创建普通用户，需要向Ubuntu-bae里添加用户默认配置文件夹`/etc/skel`,该文件夹内包含用户创建时的默认配置文件如`.bashrc、.profile`等，若没有该文件夹，在构建出的文件系统中执行`adduser`添加的用户会有各种问题，所以将宿主系统`/etc/skel`拷贝至ubuntu-base:

```
sudo cp  -R /etc/skel /mnt/etc/
```

### 2.3 进入chroot环境

**方法1**：使用原始的方法，来进入chroot环境  
**挂载和激活 /dev**：通常激活 /dev 目录下设备的方式是在 /dev 目录挂载一个虚拟文件系统（比如 tmpfs），然后允许在检测 到设备或打开设备时在这个虚拟文件系统里动态创建设备节点。这个通常是在启动过程中由 udev 完成。由于我们的ubuntu-base新系统还没有 udev，也没有被引导，有必要手动挂载和激活 /dev 这可以通过绑定挂载宿主机系统的/dev 目录来实现。绑定挂载是一种特殊的挂载模式，它允许在另外的位置创建某个目录或挂载点的镜像。运行下面的命令来实现：

```
sudo mount -v --bind /dev /mnt/dev
```

**挂载虚拟文件系统**:

```
sudo mount -vt devpts devpts /mnt/dev/pts -o gid=5,mode=620
sudo mount -vt proc proc /mnt/proc
sudo mount -vt sysfs sysfs /mnt/sys
sudo mount -vt tmpfs tmpfs /mnt/run
```

进入chroot环境：

```
chroot /mnt
```

**方法2**：使用`arch-chroot`

linux发行版archlinux提供了一个自动化chroot的脚本`arch-chroot`，包含自动配置DNS文件、自动挂载虚拟文件系统等操作，用来维护linux系统非常方便,chroot时无需挂载等操作直接执行：

```
sudo arch-chroot /mnt
```

`arch-chroot`是方法1的封装，除此之外有会对目标系统进行检测并预先配置，其源码见附录。

## 3.安装软件

### 3.1 基本配置

chroot 后为root用户，直接执行操作：  
更新软件包列表并升级：

```
apt-get update
apt-get upgrade
apt-get locales language-pack-en-base
```

设置默认使用的字符集：

```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

设置hostname

```
echo "myhostname" > /etc/hostname
```

配置/etc/hosts,文件内写入如下内容，其中`myhostname`替换为上面设置的:

```
127.0.0.1 localhost
127.0.0.1 myhostname
127.0.1.1myhostname.localdomainmyhostname
```

自动补全工具`bash-completion`，方便后续操作输入时补全.

```
apt-get install bash-completion
```

修改文件`~/.bashrc`尾，删除bash-completion前注释符`#`，并刷新环境变量，使bash-completion生效。

```
su -
```

后面的指令就可以正常使用tab补全了。

### 3.2安装内核

安装内核，Ubuntu16的代号xenial ：

```
apt-get install linux-image-generic-lts-xenial linux-headers-generic-lts-xenial
linux-modules-generic-lts-xenial 
```

安装常用工具：

```
apt install sudo openssh-server net-tools ethtool vim fish htop iputils-ping 
```

设置时区：

```
dpkg-reconfigure tzdata
```

为root设置密码：

```
passwd
```

安装uefi内核引导 grub。

```
apt-get install grub-efi-amd64-bin
```

### 3.3 安装X-window

```
apt-get   x-window-system-core  
```

```
apt-get   qt5-default libsqlite3-dev  
```

到此该系统可运行基本的图形程序。

## 4.安装各种桌面(可选)

按需ubuntu各种桌面环境，任选其一，也可同时安装多个，通过systemd选择开机启动的登录管理器来登录对应的桌面。

**GDM**\-GNOME登录管理器；

**SDDM** - 基于QML的显示管理器和KDM的后继者; 推荐用于 Plasma和 LXQt；

**XDM** - X显示管理器，支持XDMCP；

**LightDM** - 跨桌面显示管理器，可以使用任何工具包中编写的各种前端，Ubuntu16.04默认使用该管理器。

### 4.1 安装Lubuntu的定制LXDE

```
sudo apt-get install lubuntu-desktop
```

### 4.2 安装gnome桌面

其登录管理器为gdm，需要先安装xdn。

```

sudo apt-get install  gdm
sudo -y --no-install-recommends ubuntu-gnome-desktop
```

### 4.3 安装xfce桌面环境

xfce 是一款轻量级桌面.其登录管理器为xdm，需要先安装xdn。

```
apt-get install -y xdm
```

安装桌面环境

```
apt-get install -y --no-install-recommends xubuntu-desktop
```

开机启动桌面管理器。

```
systemctl enable xdm
```

### 4.4 安装mate桌面

```
sudo  apt-get install  ubuntu-mate-core
sudo apt-get install  ubuntu-mate-desktop
```

.......

## 5.arm构建简要

arm下构建流程与上面类似，可以不使用换回镜像，直接使用一个空文件夹`rootfs`，在该文件夹内进行Ubuntu rootfs构建，构建完成后再将该文件夹下的所有文件拷贝到SD卡已格式格式化后的rootfs分区内即可，也可直接挂载SD卡内的rooyfs分区操作。

目前一般arm的板子都支持从SD卡启动，同时SD卡内有两个分区，一个Fat32的启动分区内存放u-boot，另一个ext4分区为根文件系统。对于nand flash启动的板子，将roofs按文件系统类型制作烧写镜像，烧写nandflash即可。

由于arm板子的开放性，板子外设、接口等硬件资源不尽相同，不同的厂家区别很大，不像X86由硬件厂商通过BIOS（UEFI）屏蔽底层差异，兼容性强(不过现在arm这方面已改善，在arm服务器领域已与X86一样使用UEFI标准了，树莓派安装win10就是UEFI应用的一个例子，此外设备树也是借鉴了UFEI ACPI，关于UEFI，可通过https://www.zhihu.com/topic/19573354/top-answers了解)。**所以完成rootfs构建后，还需要移植好Linux内核和u-boot、设备树等。**

### 5.1下载ubuntu-base

以ubuntu16为例，[下载](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-cdimage/ubuntu-base/releases/16.04.6/release/)arm架构的ubuntu-base压缩包，可以看到有几类以后缀armXXX结尾的，它们的含义如下：

-   **ubuntu-base-xx.xx-core--arm64.tar.gz** 适用于64位arm架构，几乎所有ARMv8-A都是64位处理器，例如ARM Cortex-A53、 ARM Cortex-A57、ARM Cortex-A72、ARM Cortex-A73、RM\_Cortex-A76等。
-   **ubuntu-base-xx.xx-core-armhf.tar.gz** 适用于32位带硬浮点arm处理器，hf(hard float)，即带有浮点单元 (FPU)，主要用于ARMv7-A，例如\[ARM Cortex-A5、ARM Cortex-A7、ARM Cortex-A8、ARM Cortex-A9、ARM Cortex-A12、ARM Cortex-A15、ARM Cortex-A17。

下面以armhf为例。

### 5.3 安装qemu

```
sudo apt-get install multistrap qemu qemu-user-static binfmt-support dpkg-cross
```

### 5.4 将ubuntu-base解压

将ubuntu-base包解压到准备的rootfs文件夹,这里为`/mnt`,下面命令根据实际情况更换。

```
$sudo tar -xpvf ubuntu-base-16.04.4-base-armhf.tar.gz -C /mnt
```

拷贝`qemu-arm-static`到刚刚解压出来的目录`/mnt/usr/bin/`：

```
$sudo cp /usr/bin/qemu-arm-static /mnt/usr/bin
```

若是arm64则拷贝`qemu-aarch64-static`：

```
$sudo cp /usr/bin/qemu-aarch64-static /mnt/usr/bin
```

### 5.4 chroot操作

接下来的一切都和2.3 进入chroot环境,一致了。更新源并安装需要的软件。  
修改`/mnt/etc/apt/sources.list`，这里使用[清华源(ubuntu arm 使用的是ubuntu-ports镜像)](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/)。

```
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-backports main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
# deb http://archive.canonical.com/ubuntu xenial partner
# deb-src http://archive.canonical.com/ubuntu xenial partner

deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security main restricted
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security universe
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ xenial-security multiverse
```

```
apt-get update
apt-get install net-tools vim bash-completion ...
```

也可通过chroot直接执行某个命令，例如修改root密码,,其中`/mnt`是我们的rootfs目录：

```
$sudo chroot  /mnt  passwd
```

直接安装软件:

```
$sudo LC_ALL=C LANGUAGE=C LANG=C chroot /mnt apt-get install packagename
```

### 5.5 最后

安装内核，将内核和设备树保存到rootfs中的boot目录,即`/mnt/boot/`下。nand flash除外。

与普通文件系统烧写一致，制作烧写镜像，烧写SD卡或nandflash。

## 附录

## arch-chroot源码

请遵守相关开源协议。

```
#!/bin/bash

shopt -s extglob

# generated from util-linux source: libmount/src/utils.c
declare -A pseudofs_types=([anon_inodefs]=1
                           [autofs]=1
                           [bdev]=1
                           [binfmt_misc]=1
                           [cgroup]=1
                           [cgroup2]=1
                           [configfs]=1
                           [cpuset]=1
                           [debugfs]=1
                           [devfs]=1
                           [devpts]=1
                           [devtmpfs]=1
                           [dlmfs]=1
                           [fuse.gvfs-fuse-daemon]=1
                           [fusectl]=1
                           [hugetlbfs]=1
                           [mqueue]=1
                           [nfsd]=1
                           [none]=1
                           [pipefs]=1
                           [proc]=1
                           [pstore]=1
                           [ramfs]=1
                           [rootfs]=1
                           [rpc_pipefs]=1
                           [securityfs]=1
                           [sockfs]=1
                           [spufs]=1
                           [sysfs]=1
                           [tmpfs]=1)

# generated from: pkgfile -vbr '/fsck\..+' | awk -F. '{ print $NF }' | sort
declare -A fsck_types=([cramfs]=1
                       [exfat]=1
                       [ext2]=1
                       [ext3]=1
                       [ext4]=1
                       [ext4dev]=1
                       [jfs]=1
                       [minix]=1
                       [msdos]=1
                       [reiserfs]=1
                       [vfat]=1
                       [xfs]=1)

out() { printf "$1 $2\n" "${@:3}"; }
error() { out "==> ERROR:" "$@"; } >&2
warning() { out "==> WARNING:" "$@"; } >&2
msg() { out "==>" "$@"; }
msg2() { out "  ->" "$@";}
die() { error "$@"; exit 1; }

ignore_error() {
  "$@" 2>/dev/null
  return 0
}

in_array() {
  local i
  for i in "${@:2}"; do
    [[ $1 = "$i" ]] && return 0
  done
  return 1
}

chroot_add_mount() {
  mount "$@" && CHROOT_ACTIVE_MOUNTS=("$2" "${CHROOT_ACTIVE_MOUNTS[@]}")
}

chroot_maybe_add_mount() {
  local cond=$1; shift
  if eval "$cond"; then
    chroot_add_mount "$@"
  fi
}

chroot_setup() {
  CHROOT_ACTIVE_MOUNTS=()
  [[ $(trap -p EXIT) ]] && die '(BUG): attempting to overwrite existing EXIT trap'
  trap 'chroot_teardown' EXIT

  chroot_add_mount proc "$1/proc" -t proc -o nosuid,noexec,nodev &&
  chroot_add_mount sys "$1/sys" -t sysfs -o nosuid,noexec,nodev,ro &&
  ignore_error chroot_maybe_add_mount "[[ -d '$1/sys/firmware/efi/efivars' ]]" \
      efivarfs "$1/sys/firmware/efi/efivars" -t efivarfs -o nosuid,noexec,nodev &&
  chroot_add_mount udev "$1/dev" -t devtmpfs -o mode=0755,nosuid &&
  chroot_add_mount devpts "$1/dev/pts" -t devpts -o mode=0620,gid=5,nosuid,noexec &&
  chroot_add_mount shm "$1/dev/shm" -t tmpfs -o mode=1777,nosuid,nodev &&
  chroot_add_mount run "$1/run" -t tmpfs -o nosuid,nodev,mode=0755 &&
  chroot_add_mount tmp "$1/tmp" -t tmpfs -o mode=1777,strictatime,nodev,nosuid
}

chroot_teardown() {
  if (( ${#CHROOT_ACTIVE_MOUNTS[@]} )); then
    umount "${CHROOT_ACTIVE_MOUNTS[@]}"
  fi
  unset CHROOT_ACTIVE_MOUNTS
}

try_cast() (
  _=$(( $1#$2 ))
) 2>/dev/null

valid_number_of_base() {
  local base=$1 len=${#2} i=

  for (( i = 0; i < len; i++ )); do
    try_cast "$base" "${2:i:1}" || return 1
  done

  return 0
}

mangle() {
  local i= chr= out=
  local {a..f}= {A..F}=

  for (( i = 0; i < ${#1}; i++ )); do
    chr=${1:i:1}
    case $chr in
      [[:space:]\\])
        printf -v chr '%03o' "'$chr"
        out+=\\
        ;;
    esac
    out+=$chr
  done

  printf '%s' "$out"
}

unmangle() {
  local i= chr= out= len=$(( ${#1} - 4 ))
  local {a..f}= {A..F}=

  for (( i = 0; i < len; i++ )); do
    chr=${1:i:1}
    case $chr in
      \\)
        if valid_number_of_base 8 "${1:i+1:3}" ||
            valid_number_of_base 16 "${1:i+1:3}"; then
          printf -v chr '%b' "${1:i:4}"
          (( i += 3 ))
        fi
        ;;
    esac
    out+=$chr
  done

  printf '%s' "$out${1:i}"
}

optstring_match_option() {
  local candidate pat patterns

  IFS=, read -ra patterns <<<"$1"
  for pat in "${patterns[@]}"; do
    if [[ $pat = *=* ]]; then
      # "key=val" will only ever match "key=val"
      candidate=$2
    else
      # "key" will match "key", but also "key=anyval"
      candidate=${2%%=*}
    fi

    [[ $pat = "$candidate" ]] && return 0
  done

  return 1
}

optstring_remove_option() {
  local o options_ remove=$2 IFS=,

  read -ra options_ <<<"${!1}"

  for o in "${!options_[@]}"; do
    optstring_match_option "$remove" "${options_[o]}" && unset 'options_[o]'
  done

  declare -g "$1=${options_[*]}"
}

optstring_normalize() {
  local o options_ norm IFS=,

  read -ra options_ <<<"${!1}"

  # remove empty fields
  for o in "${options_[@]}"; do
    [[ $o ]] && norm+=("$o")
  done

  # avoid empty strings, reset to "defaults"
  declare -g "$1=${norm[*]:-defaults}"
}

optstring_append_option() {
  if ! optstring_has_option "$1" "$2"; then
    declare -g "$1=${!1},$2"
  fi

  optstring_normalize "$1"
}

optstring_prepend_option() {
  local options_=$1

  if ! optstring_has_option "$1" "$2"; then
    declare -g "$1=$2,${!1}"
  fi

  optstring_normalize "$1"
}

optstring_get_option() {
  local opts o

  IFS=, read -ra opts <<<"${!1}"
  for o in "${opts[@]}"; do
    if optstring_match_option "$2" "$o"; then
      declare -g "$o"
      return 0
    fi
  done

  return 1
}

optstring_has_option() {
  local "${2%%=*}"

  optstring_get_option "$1" "$2"
}

dm_name_for_devnode() {
  read dm_name <"/sys/class/block/${1#/dev/}/dm/name"
  if [[ $dm_name ]]; then
    printf '/dev/mapper/%s' "$dm_name"
  else
    # don't leave the caller hanging, just print the original name
    # along with the failure.
    print '%s' "$1"
    error 'Failed to resolve device mapper name for: %s' "$1"
  fi
}

fstype_is_pseudofs() {
  (( pseudofs_types["$1"] ))
}

fstype_has_fsck() {
  (( fsck_types["$1"] ))
}


usage() {
  cat <<EOF
usage: ${0##*/} chroot-dir [command]

    -h                  Print this help message
    -u <user>[:group]   Specify non-root user and optional group to use

If 'command' is unspecified, ${0##*/} will launch /bin/bash.

Note that when using arch-chroot, the target chroot directory *should* be a
mountpoint. This ensures that tools such as pacman(8) or findmnt(8) have an
accurate hierarchy of the mounted filesystems within the chroot.

If your chroot target is not a mountpoint, you can bind mount the directory on
itself to make it a mountpoint, i.e. 'mount --bind /your/chroot /your/chroot'.

EOF
}

chroot_add_resolv_conf() {
  local chrootdir=$1 resolv_conf=$1/etc/resolv.conf

  [[ -e /etc/resolv.conf ]] || return 0

  # Handle resolv.conf as a symlink to somewhere else.
  if [[ -L $chrootdir/etc/resolv.conf ]]; then
    # readlink(1) should always give us *something* since we know at this point
    # it's a symlink. For simplicity, ignore the case of nested symlinks.
    resolv_conf=$(readlink "$chrootdir/etc/resolv.conf")
    if [[ $resolv_conf = /* ]]; then
      resolv_conf=$chrootdir$resolv_conf
    else
      resolv_conf=$chrootdir/etc/$resolv_conf
    fi

    # ensure file exists to bind mount over
    if [[ ! -f $resolv_conf ]]; then
      install -Dm644 /dev/null "$resolv_conf" || return 1
    fi
  elif [[ ! -e $chrootdir/etc/resolv.conf ]]; then
    # The chroot might not have a resolv.conf.
    return 0
  fi

  chroot_add_mount /etc/resolv.conf "$resolv_conf" --bind
}

while getopts ':hu:' flag; do
  case $flag in
    h)
      usage
      exit 0
      ;;
    u)
      userspec=$OPTARG
      ;;
    :)
      die '%s: option requires an argument -- '\''%s'\' "${0##*/}" "$OPTARG"
      ;;
    ?)
      die '%s: invalid option -- '\''%s'\' "${0##*/}" "$OPTARG"
      ;;
  esac
done
shift $(( OPTIND - 1 ))

(( EUID == 0 )) || die 'This script must be run with root privileges'
(( $# )) || die 'No chroot directory specified'
chrootdir=$1
shift

[[ -d $chrootdir ]] || die "Can't create chroot on non-directory %s" "$chrootdir"

if ! mountpoint -q "$chrootdir"; then
  warning "$chrootdir is not a mountpoint. This may have undesirable side effects."
fi

chroot_setup "$chrootdir" || die "failed to setup chroot %s" "$chrootdir"
chroot_add_resolv_conf "$chrootdir" || die "failed to setup resolv.conf"

chroot_args=()
[[ $userspec ]] && chroot_args+=(--userspec "$userspec")
echo "${chroot_args[@]}"
echo  "$chrootdir"
echo  "$@"
SHELL=/bin/bash unshare --fork --pid chroot "${chroot_args[@]}" -- "$chrootdir" "$@"
```
