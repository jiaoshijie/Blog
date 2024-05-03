---
title: "Linux 配置文件读取规则"
date: 2024-05-03T19:13:32+08:00
tags: ["X", "Linux"]
categories: ""
author: "Jiao shijie"
draft: true
hidemeta: true
math: false
---

## bash 或 zsh 登入时读取的配置文件
1. shell 登入分为两种 `login shell` `non login shell`
2. 只有 `login shell` 登入时读取的配置文件 `/etc/profile` `~/.bash_profile` ~~~/.bash_login~~ ~~~/.profile~~
  - `/etc/profile` 是全局配置文件所有用户都会读取, `~/.bash_profile` 是用户的配置文件只有当前用户会读取.
  - 后面的用删除线划掉的也是只有 `login shell` 才会读取的用户配置文件, 但只有在 `~/.bash_profile` 文件不存在时才会读取
  - `~/.bash_profile` ~~~/.bash_login~~ ~~~/.profile~~ 这三个配置文件只会读取一个, 优先级为 `~/.bash_profile` > `~/.bash_login` > `~/.profile` 所以后两个基本不会用到.
  - `zsh` 的 `login shell` 配置文件为 `/etc/zsh/zprofile` `~/.zprofile` 更具体的zsh说明可以看[这里](https://wiki.archlinux.org/index.php/Zsh)
3. 两种登入都会读取的配置文件 `/etc/bashrc` `~/.bashrc`
  - `/etc/bashrc` 全局配置文件, 并且会调用 `/etc/profile` 来设置 PATH umask等
  - Arch 发行版好像没有 `/etc/bashrc` 文件, 用 `/etc/bash.bashrc` 来替代了
  - `~/.bashrc` 用户配置文件, 两种登入都会读取, 没什么可说的

## X-Window 启动时读取的配置文件
1. xserver 启动配置文件 `~/.xserverrc` `/etc/X11/xinit/xserverrc`
  - 由 `X` 程序执行
  - 就是设置一些启动参数什么的, 比如: 允不允许网络登录, 监不监听tcp端口什么的
2. xserver 设备配置文件 `/ect/X11/xorg.conf` `/etc/X11/xorg.conf.d/*`
  - 设备的设置选项, 比如: 显卡、触控板、鼠标等[更详细的说明可以看这里](https://wiki.archlinux.org/index.php/Xorg)
  - `/etc/X11/xorg.conf.d/*` 文件 `30-touchpad.conf` 前面的数字是读取顺序(10会在30之前被读取)
3. xclient 启动配置文件 `~/.xinitrc` `/etc/X11/xinit/xinitrc`
  - 由 `xinit` 程序执行

### X-window 其他配置文件
1. `Xsession` `~/.xsession` `~/.xsessionrc`
  - 一些登入管理器会使用Xsession进行登入
  ```
  GDM - /etc/gdm/Xsession
  LightDM - /etc/lightdm/Xsession
  LXDM - /etc/lxdm/Xsession
  SDDM - /usr/share/sddm/scripts/Xsession
  ```
  - 有一些登录管理器像XDM就默认不使用Xsession而是使用 `~/.xsession` 来进行登入的设置, 具体可以看[这里](https://wiki.archlinux.org/index.php/XDM)
  - `~/.xsessionrc` 只有debian分支才会使用的一个配置文件而且好像还要使用debian默认的登入管理器才会用到, 具体可以看看[debian官方说明](https://wiki.debian.org/Xsession)和[论坛介绍](https://unix.stackexchange.com/questions/281858/difference-between-xinitrc-xsession-and-xsessionrc)
2. `/etc/xprofile` `~/.xprofile`
  - 默认情况下只有一些 `display manager(DM)` 才会读取的配置文件display manager(DM) 登入管理器(GDM、LightDM、LXDM、SDDM)
  - 但也可以配置使(startx、xinit、XDM、和其他使用 `~/.xsession` 或 `~/.xinitrc` 的DM)读取 `xprofile`
  - 在 .xsession 或 .xinitrc 中添加一下内容
  ```
  # Make sure this is before the 'exec' command or it won't be sourced.
  [ -f /etc/xprofile ] && . /etc/xprofile
  [ -f ~/.xprofile ] && . ~/.xprofile
  ```
3. `~/.Xresources` `~/.Xdefaults`
  - 这两个文件都是用户级别的配置文件，功能基本相同. 通过 `xrdb` 程序加载.
  - 设置终端一些参数, 设置DPI, 字体, 设置一些底层的应用程序(xorg-xclock, xpdf, urxvt)等.
  - 其实, xrdb 默认是不会加载任何配置文件的. 所以这两个配置文件的名称只是一种约定, 现在都使用 `~/.Xresources` 来代替 `~/.Xdefaults`
4. `~/.Xmodmap`
  - 由 `xmodmap` 程序执行
  - 设置键盘的键位映射
5. xdg(X Desktop Group)目录
  - 一些 `x application` 的配置文件
  - `/etc/xdg/autostart/` 一些支持XDG的桌面环境会自动启动程序的目录
6. `~/.Xauthority` 文件
  - startx 启动脚本中默认会创建并使用这个文件(但是可以取消通过 `enable_xauth` 变量)
  - DM 也会创建并使用这个文件如果创建不成功会无法登入进系统，一直返回DM界面
  - `ssh -Y` 通过网络来链接客户端会创建并使用这个文件当启动GUI程序时, 会通过这个文件来区分本地用户还是网络用户
