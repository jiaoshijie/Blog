---
title: "Linux_Little_Tricks"
date: 2024-05-03T19:48:28+08:00
tags: ["linux"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

# Linux Little Tricks

- [data model LLP64 and LP64](https://en.wikipedia.org/wiki/64-bit_computing#64-bit_data_models)

## 搜索引擎小技巧

- `cats and dogs` Result about cats or dogs
- `"cats and dogs"` Result about exact term "cats and dogs"
- `cats -dogs` Fewer dogs in result
- `cats +dogs` More dogs in result
- `cats filetype:pdf` PDFs about cats
  * Supported types: pdf, doc(x), xls(x), ppt(x), html
- `*` 模糊
- `....` 范围
- `intitle:标题`
- `inurl:网址`
- `related:相关类型`
- `define:word`
- `etymology:"word,"`
- `joker site:drive.google.com`

## Proxy

- Browser: `SwitchyOmega`
- gfwlist: `https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt`
- [Terminal Proxy](https://about.gitlab.com/blog/2021/01/27/we-need-to-talk-no-proxy/)

## CapsLock -> Ctrl

### X11

- `xev` 展示按下按键的信息
- `pacman -S xorg-xmodmap`: `xmodmap` `xmodmap -pke` `xmodmap -pke > ~/.Xmodmap`
- 使用 [`remaps`](https://github.com/jiaoshijie/dots/blob/main/scripts/bin/remaps) 脚本

### Wayland

- `dconf write /org/gnome/desktop/input-sources/xkb-options "['caps:ctrl_modifier']"`

## 触控板设置

- [xorg 使用 libinput 配置触控板](https://wiki.archlinux.org/title/Libinput)
- [dwm](https://dwm.suckless.org/) 默认触控板无法使用单击, 双击等
- 在 `xorg.conf.d` 目录下添加 `30-touchpad.conf` 配置文件, 并添加以下配置
  - `30` 表示执行顺序 (10 比 30 先执行)
  - `.conf` 后缀 arch wiki 说必须加上不可以省略

```bash
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
    Option "NaturalScrolling" "true"
EndSection
```

## 多显示器设置

- [Xorg Multiple Monitors](https://wiki.gentoo.org/wiki/Xorg/Guide#Multiple_monitors)

X11 默认多显示器是相同输出, `xrandr` 可以用来查看显示器输出端口的信息, 可以通过两种方式设置

1. `xrandr --output DP1 --auto --right-of eDP1` 可以设置显示器显示不同的内容
2. 设置配置文件 `/etc/X11/xorg.conf.d/40-monitor.conf`

```bash
Section "Device"
  Identifier "Intel Corporation HD Graphics 630"  # 视频输出的显卡名，可以通过 `lspci -v` 查看
  Option "Monitor-eDP1" "eDP1 screen"
  Option "Monitor-DP1" "DP1 screen"
EndSection

Section "Monitor"
  Identifier "DP1 screen"
  Option "RightOf" "eDP1 screen"
EndSection

Section "Monitor"
  Identifier "eDP1 screen"
EndSection
```

- 设置 Xorg 多显示器显示相同的内容
  + `xrandr --output DP1 --auto --same-as eDP1`
  + `--mode` 可以设置分辨率

## Archlinux install Latex

- `pacman -S texlive-most`
  + [x] `texlive-most` is a package group, includes `texlive-core texlive-bin`
  + [ ] `texlive-lang` 添加除英文外的其他语言的支持, 不推荐安装
- `pdftex '\empty Hello World!\bye'` 查看是否安装成功
- `texdoc` 查看帮助文档的命令

## NetworkManager

- [nmcli usage](https://wiki.archlinux.org/title/NetworkManager#nmcli_examples)
- [nmcli setting static ip](https://linuxhint.com/configure-static-ip-address-fedora/)

## 设置 grub 主题

1. 复制 grub2 主题到 `/boot/grub/themes`
2. `vim /etc/default/grub` 修改 `GRUB_THEME=/boot/grub/themes/{theme_name}/theme.txt`
3. `grub-mkconfig -o /boot/grub/grub.cfg` 重新创建 grub 配置

## archlinux using 32位 gcc

首先, 修改 `/etc/pacman.conf` 文件

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

安装 `pacamn -S multilib-devel lib32-glibc lib32-gcc-libs`

## 使用 cronie 设置定时任务

- [scheduling cron jobs with crontab](https://linuxize.com/post/scheduling-cron-jobs-with-crontab/)
- [Understanding Crontab in Linux With Examples](https://linuxhandbook.com/crontab/)

## tmux 使用 ssh 连接，断开连接后 tmux session 也会关闭的问题

- 默认应该是不会发生这种情况的，但如果发生了看下面的解决办法
- 编辑 `/etc/systemd/logind.conf` 文件
- 设置 `KillExcludeUsers=root MY_USERNAME` 和 `KillUserProcess=no`
- 产生问题的原因应该是 optimus manager 在这个文件中 `/usr/lib/systemd/logind.conf.d/*.conf` 设置了 `KillUserProcess=yes`
- [tmux server being killed after logging out](https://bbs.archlinux.org/viewtopic.php?id=294348)

## 使用 laptop 时关闭合盖后依然保持运行

- 编辑 `/etc/systemd/logind.conf` 文件
- 设置 `HandleLidSwitch=ignore` 变量重启系统即可
- [how to disable auto suspend when i close laptop lid](https://unix.stackexchange.com/questions/52643/how-to-disable-auto-suspend-when-i-close-laptop-lid)

## systemd

- `.service` 文件位置: `/usr/lib/systemd/system/` `/usr/lib/systemd/user/` `~/.config/systemd/user/`
- [how-to-set-up-an-automatic-restart-of-a-background-ssh-tunnel](https://stackoverflow.com/questions/6758897/how-to-set-up-an-automatic-restart-of-a-background-ssh-tunnel)
- [tunnelling-with-ssh-different-approaches-and-tips](https://blog.digitalis.io/tunnelling-with-ssh-different-approaches-and-tips-a964e4d6a321)

## file permissions

- files
  * Read – Can view or copy file contents
  * Write – Can modify file content
  * Execute – Can run the file (if its executable)
- directories
  * Read – Can list all files and copy the files from directory
  * Write – Can add or delete files into directory (needs execute permission as well)
  * Execute – Can enter the directory

## locale problem: not found files

- Arch: **See Installation Guide**
- Fedora: `dnf install langpacks-zh_CN`
- Ubuntu:
  + `cd /usr/share/locales`
  + `./install-language-pack zh_CN.UTF-8`
  + `dpkg-reconfigure locales`

## X11 Debug a Window Manager

- use `Xephyr :1` to start a new DISPLAY server
- use `DISPLAY=:1 {gui_command}` to start `gui_command` in the new server

## ubuntu using x11 instead of wayland

- `vim /etc/gdm3/custom.conf`
- uncomment this line `#WaylandEnable=false`

## ubuntu disable showing icons in desktop

- `sudo apt remove gnome-shell-extension-desktop-icons-ng -y`
- ref: [How disable show folder in desktop ubuntu?](https://askubuntu.com/questions/1407605/how-disable-show-folder-in-desktop-ubuntu)

## VirtualBox

### Arch Linux Install

- `pacman -S virtualbox`
  * 使用`virtualbox-host-modules-arch`内核依赖
- 创建虚拟机有问题就重新加载内核执行命令 `sudo vboxreload`
- [ ] Install extpack
  * download the extension manually
  * `sudo VBoxManage extpack install <.vbox-extpack>`

### VirtualBox in windows 11 are so slow

- [fix-virtualbox-running-very-slow-in-windows-10-11](https://www.wintips.org/fix-virtualbox-running-very-slow-in-windows-10-11/)

### `<` `<<` `<<<` `<()`

- [input redirections explation](https://linuxhandbook.com/here-input-redirections/?ref=lhb-linux-digest-newsletter)
- [gnu redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)

- `<`: The input redirection `cat < README.md`
- `<<`: The here document `cat > output.txt <<EOF` the EOF is an DELIMITER, when stdin recives the DELIMITER, the stdin will close
- `<<<`: The here string `cat <<< "Demo text"` input one line of stings to a script or a command.
- `<()`: redirection command output to another command `cat <(echo "Demo text")`
