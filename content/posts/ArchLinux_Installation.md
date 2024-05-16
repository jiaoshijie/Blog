---
title: "ArchLinux_Installation"
date: 2024-05-03T19:46:54+08:00
tags: ["linux", "arch"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

## 一. Basic Install

- [制作启动盘](https://wiki.archlinux.org/title/USB_flash_installation_medium)

### 1. 系统启动方式

1. 输入 `ls /sys/firmware/efi/efivars` 查看系统的启动方式
2. 如果有输出，则为UEFI启动

### 2. 网络连接

`ip link` 查看网卡的名称等信息

#### 有线网络

使用 `dhcpcd` 进行有线网络连接, 和激活无限网络dhcp功能

#### 无线网络

1. 使用 `rfkill`(Radio Frequency Kill) 检查 wifi 是否被禁用

```bash
$ rfkill list

0: phy0: Wireless LAN
Soft blocked: yes  // yes 为禁用, no 为未禁用(软件禁用)
Hard blocked: yes  // yes 为禁用, no 为未禁用(硬件禁用)
```

2. 如果被禁用使用 `rfkill unblock wifi` 取消禁用
3. 使用 `iwctl` 进行无线网络连接

```bash
iwctl$ help
iwctl$ device list
iwctl$ station device(name) scan
iwctl$ station device(name) get-networks
iwctl$ station device(name) connect SSID(name)
```

4. 使用 `ping archlinux.org` 检查网络是否可用

### 3. 设置系统时间 TODO

1. `timedatectl set-ntp true` 确保时钟正确, 正确则没有输出
- To check the service status, use `timedatectl status`

### 4. 磁盘分区

#### 磁盘规划

- **EFI**: 100M
- **/**: 100G 或者 一整块硬盘(如果是多硬盘的话)
- **/home/{username}/Downloads**: 剩下的空间 或 其他整块硬盘
- **swap**: 不设置 swap 分区

#### 分区和格式化

1. `lsblk` 查看系统硬盘情况
2. `fdisk /dev/{disk}` 用来进行磁盘分区

```bash
fdisk$ m # 查看帮助
fdisk$ g # 创建一个 GPT 分区，用来作为 EFI 分区
fdisk$ # ...
fdisk$ n # 用来创建其他分区
fdisk$ # ...
fdisk$ l # 查看支持的所有分区类型 ef(EFI 分区类型) 83(Linux 分区类型)
fdisk$ t # 来更改创建分区的分区类型
fdisk$ # NOTE: 将 GPT 分区改为 EFI 分区类型, 其余分区改为 Linux 分区类型
fdisk$ w # 保存退出
fdisk$ q # 取消修改退出
```

3. 使用 `mkfs.{format} /dev/{disk}` 格式化磁盘
  - `mkfs.vfat` 格式化 EFI 分区
  - `mkfs.ext4` 格式化其他分区
  - NOTE: 不适用 SWAP 分区
4. 将根目录挂在到 `/mnt` 目录下, 将 EFI 分区挂载到 `/mnt/efi` 其余目录挂在到 `/mnt/*` 目录下

### 5. 系统下载与安装

- [ ] FIXME: `vim /etc/pacman.d/mirrorlist` 设置安装源, 可以先使用国内的源，后面在去掉就行。
  + `Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch`
  + `Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch`
- [ ] 安装系统基本包 `pacstrap /mnt base base-devel linux linux-headers linux-firmware man-db man-pages networkmanager vim bash-completion`
  + [ ] `base`
  + [ ] `base-devel`
  + [ ] `linux`
  + [ ] `linux-headers`
  + [ ] `linux-firmware`
  + [ ] `man-db`
  + [ ] `man-pages`
  + [ ] `networkmanager`
  + [ ] `vim`
  + [ ] `bash-completion`

### 6. 生成系统自动挂载分区文件(fstab文件)

1. `genfstab -L /mnt >> /mnt/etc/fstab`
2. `cat /mnt/etc/fstab` 检查写入是否正确

### 7. 将 root 目录由安装镜像系统切换到要安装的系统

- `arch-chroot /mnt`

### 8. 时间区域设置

1. `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
2. [ ] 使用hwclock创建`/etc/adjtime` `hwclock --systohc`

### 9. 系统本地化设计

1. 设置本地语言 `vim /etc/locale.gen` 找到文件中的 `en_US.UTF-8 UTF-8` `zh_CN.UTF-8 UTF-8`
2. `locale-gen` 加载设置的语言项
3. 打开文件 `vim /etc/locale.conf` 并在里面输入一行 `LANG=en_US.UTF-8`
4. 设置主机名称 `vim /etc/hostname` 然后输入自己定义的主机名称(`sakura`)
5. 设置本机地址 `vim /etc/hosts` 添加以下内容

```bash
127.0.0.1 localhost
::1       localhost
```

### 10. 设置 root 密码

- `passwd root`

### 11. 安装 ucode

- intel-CPU `pacman -S intel-ucode`
- AMD-CPU `pacman -S amd-ucode`

### 12. bootloader

- [Arch GRUB wiki](https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks#Combining_the_use_of_UUIDs_and_basic_scripting)
- `pacman -S grub efibootmgr`
- [ ] `pacman -S os-prober` 用来检查 windows 系统, 单系统可以不装
- `grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB`
- `vim /boot/grub/grub.cfg` 修改`GRUB_CMDLINE_LINUX_DEFAULT` 变量中的参数
  + 去掉 `quiet`
  + `log level` 由 3 变为 5, 方便排错
  + [ ] 加入 `nowatchdog` 可以加快开机启动速度
- [ ] `grub-mkconfig -o /boot/grub/grub.cfg`
- 基本系统安装完成 `exit` `umount -R /mnt` `reboot`

## 二. GUI

### 1. 创建一个普通用户

- `pacman -S sudo`
- `useradd -m -g users -G wheel -s /bin/bash {username}`
  * `-m` 创建用户的home目录
  * `-g` 指定初始的用户组
  * `-G` 指定扩展的用户组
  * `-s` 指定默认的shell
- 设置密码 `passwd {username}`
- 设置`vim /etc/sudoers`去掉注释`%wheel ALL=(ALL) ALL`

### 2. X11

- `pacman -S xorg xorg-xinit xorg-xrandr`
  + [Xorg arch wiki](https://wiki.archlinux.org/title/Xorg)
  + [xorg](https://archlinux.org/groups/x86_64/xorg/) is a package group
  + [xinit](https://wiki.archlinux.org/title/Xinit) 可以手动的启动 Xorg display server。
  + [xorg-xrandr](https://wiki.archlinux.org/title/Xrandr)
- `pacman -S xf86-input-libinput`
  + [xf86-input-libinput](https://archlinux.org/packages/extra/x86_64/xf86-input-libinput/)

### 3. 声卡驱动

- [ ] `alsa-utils` 配置复杂
- [x] `pulseaudio` 配置简单
  + `pulseaudio-bluetooth` pulseaudio-alsa
- [ ] `pipewire` 新框架(使用有问题)
  + `pipewire-media-session`  A very simple session manager
  + `wireplumber pipewire-pulse` pipewire-alsa pipewire-jack

一些前端配置工具 `pamixer pulsemixer pavucontrol`

### 4. 笔记本电源管理 TODO: 目前没有使用

- `tlp tlp-rdw` simply install it and forget it
  + `sudo systemctl enable tlp.service`
  + `sudo systemctl unmask systemd-rfkill.service`
  + `sudo systemctl unmask systemd-rfkill.socket`

### 5. 下载 32 位的程序及添加 Chaotic-AUR(将AUR中的包编译成二进制下载)

修改 `vim /etc/pacman.conf`, 取消注释

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

- [添加 Chaotic-AUR 仓库](https://aur.chaotic.cx/)

### 6. 设置 DNS

修改 `vim /etc/resolv.conf`, 删除已有内容, 添加以下内容

```bash
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
nameserver 8.8.4.4
nameserver 2001:4860:4860::8844
```

- `chattr +i /etc/resolv.conf` 防止该文件被重写
  + `chattr` 可以设置文件的属性
  + `+i` 为指定文件增加 i 属性，i 属性表示不可修改
  + `lsattr` 可以查看一个文件的属性

### 7. 显卡驱动

- [显卡驱动文档](https://wiki.archlinux.org/title/Xorg#Driver_installation)
- `lspci -v | grep -A1 -e VGA -e 3D` 验证显卡信息
  + [intel](https://wiki.archlinux.org/title/Intel_graphics#Installation) `mesa lib32-mesa vulkan-intel lib32-vulkan-intel`
  + [ ] [AMD](https://wiki.archlinux.org/title/AMDGPU#Installation) `xf86-video-amdgpu mesa lib32-mesa vulkan-radeon lib32-vulkan-radeon libva-mesa-driver lib32-libva-mesa-driver mesa-vdpau lib32-mesa-vdpau`

#### Nvidia 显卡驱动

- [NVIDIA](https://wiki.archlinux.org/title/NVIDIA) `nvidia nvidia-settings lib32-nvidia-utils`

#### 双显卡

- [NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus)
- [ ] 安装 `mesa-demos` 后，可使用 `glxinfo | grep "OpenGL renderer"` 来查看当前使用的显卡。

##### Bumblebee

该方式是在通电情况下, 使用集成显卡进行显示管理, 而独立显卡进行复杂的渲染工作, 并将要显示的内容传送到集成显卡上进行显示, 当使用电池时将关闭独立显卡.

 1. 安装显卡驱动
 2. 安装bumblebee来管理独立显卡的使用 `pacman -S bumblebee bbswitch`
 3. 修改配置文件 `/etc/bumblebee/bumblebee.conf` 将Driver的值设置为 `nvidia`, 来让其使用nvidia驱动, 然后将PMMethod的值设置为 `bbswitch` 让它使用bbswitch来管理显卡
 4. 将要使用Bumblebee的用户添加到bumblebee组中，`gpasswd -a username bumblebee`.
 5. 并将bumblebeed服务设为开机启动, `systemctl enable bumblebeed` 重启计算机.
 6. 需要mesa-demos软件包, 使用 `optirun glxgears -info` 查看bumblebee是否正常工作, 如果有显示则代表正常工作.

当使用CUDA程序时可能bumblebee不会启动(如: python-pytorch)可以通过以下操作来进行使用CUDA程序:
  - 使用 `sudo tee /proc/acpi/bbswitch <<< ON` 来启动nvidia
  - 使用 `sudo rmmod nvidia nvidia_uvm nvidia_modeset` 和 `sudo tee /proc/acpi/bbswitch <<< OFF` 来关闭nvidia在使用完CUDA程序之后.

##### optimus-manager (recommended)

- `paru -S optimus-manager`
- `sudo systemctl enable optimus-manager`
- [设置optimus-manager在.xinitrc中使用](https://dev.to/snikhill/optimus-manager-on-arch-linux-1589)
- [optimus-manager-github](https://github.com/Askannz/optimus-manager/wiki/FAQ,-common-issues,-troubleshooting)

### 8. 安装字体和输入法

#### 英文字体

- [X] `ttf-linux-libertine ttf-dejavu ttf-ubuntu-font-family noto-fonts`
- [ ] `noto-fonts-extra`

#### 中文字体

- [Chinese fonts List](https://wiki.archlinux.org/index.php/Localization/Chinese)
- [X] `wqy-microhei` `noto-fonts-cjk(Chinese, Japanese, Korean fonts)`

#### 等宽字体

- [programming fonts review](https://www.programmingfonts.org/)
- [X] programming ligatures: `ttf-jetbrains-mono` `ttf-cascadia-code` `ttf-monaspace-variable`
- [X] `adobe-source-code-pro-fonts` `ttf-hack`

#### 图标字体

- [X] `ttf-sourcecodepro-nerd`

#### emoji 字体

- [X] `noto-fonts-emoji` `ttf-joypixels`

#### Linux 本地化设置

- [archlinux字体配置](https://wiki.archlinux.org/index.php/Localization/Simplified_Chinese)
- `en_US.UTF-8` 和 `zh_CN.UTF-8` 的区别, 程序的默认语言不同, 但字符集都是UTF-8, 可以显示的文字相同.
- `locale.conf` 文件命名格式 `<语言>_<地区名>.<字符编码名称>`

和本地化有关的变量如下，优先级 `LC_ALL > LC_*> LANG`

```bash
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"  # 用于字符分类和字符串处理, 控制所有字符的处理方式, 包括字符编码, 字符是单字节还是多字节, 如何打印等
LC_NUMERIC="en_US.UTF-8"  # 数字显示
LC_TIME="en_US.UTF-8"  # 时间显示格式
LC_COLLATE="en_US.UTF-8"  # 比较和排序习惯
LC_MONETARY="en_US.UTF-8"  # 货币单位
LC_MESSAGES="en_US.UTF-8"  # 用于控制程序输出时所使用的语言, 主要是提示信息, 错误信息, 状态信息, 标题, 标签, 按钮和菜单等
LC_PAPER="en_US.UTF-8"  # 默认纸张尺寸大小
LC_NAME="en_US.UTF-8"  # 姓名书写方式
LC_ADDRESS="en_US.UTF-8"  # 地址书写方式
LC_TELEPHONE="en_US.UTF-8"  # 电话号码书写方式
LC_MEASUREMENT="en_US.UTF-8"  # 度量单位表达方式
LC_IDENTIFICATION="en_US.UTF-8"  # 对 locale 自身包含信息的概述
LC_ALL=
```

#### 安装输入法

- [x] `fcitx5-im fcitx5-chinese-addons fcitx5-pinyin-zhwiki`
- 将以下配置放入 `~/.bashrc` 中如果使用 `bash`。

```bash
export XMODIFIERS=@im=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export INPUT_METHOD=fcitx
export SDL_IM_MODULE=fcitx
export GLFW_IM_MODULE=ibus
```

### 9. dwm

- [x] 安装 dwm
