---
title: "Bluetooth Setup On Arch Linux"
date: 2024-08-04T00:02:32+08:00
tags: ["linux"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

- [Xbox series x controller](https://www.reddit.com/r/linux_gaming/comments/lenhbc/xbox_series_x_controller_with_linux/)

## 首先安装蓝牙服务

`pacman -S bluez bluez-utils pulseaudio-bluetooth`

- `bluez` 提供bluetooth服务
- `bluez-utils` 提供bluetoothctl工具 `bluez-utils-compat`(开机后无法NOT Default controller available)使用这个包据说可以修复这个问题.
- (optional)`blueman`显示systray
- `pulseaudio-bluetooth` 可以连接蓝牙耳机

## 启动连接设备

- `sudo systemctl start bluetooth.service`开启bluetooth服务. `sudo systemctl enable bluetooth.service`设置为开机启动.
- 然后使用`bluetoothctl`进行蓝牙的连接.
- 将用户添加到`lp`用户组中, 添加方法`usermod -G Groupname Username`, 或直接修改`/etc/group`文件.

### 首次启动蓝牙

- 使用`help`查看帮助.
- 首先选择设备`select MAC_address`
- 然后使用`power on`开启蓝牙设备
  * 如果开启失败检查是否是蓝牙设备被block了. 使用`rfkill list`观察.
  * 如果是可以`rfkill unblock all` or `connmanctl enable bluetooth`来取消所有的被block的设备. 详细使用`man rfkill`.
- 然后使用`devices`查看可以使用的设备.
- (optional)如果devices没有可用的设备. 使用`scan on`来扫描附近可连接的蓝牙设备.
- (optional)`agent on`或选择其他的模式(DisplayOnly KeyboardDisplay NoInputNoOutput DisplayYesNO KeyboardOnly off on)如果你清楚的话.
- `pair MAC_address`开始配对设备.
- 如果设备没有PIN, 可能还需要手动`trust MAC_address`在重新连接之前.
- `connect MAC_address`来连接蓝牙设备.
  * 如果连接失败, 可能是`pulseaudio-bluetooth`没有安装. 具体使用`systemctl status bluetooth.service`查看错误信息.
  * 然后在`/etc/pulse/system.pa`中加入以下内容:

```config
### Enable bluetooth daemon
load-module module-bluetooth-policy
load-module module-bluetooth-discover
```

### 之后每次启动bluetoothctl

- `sudo systemctl start bluetooth.service` 如果不想每次reboot输入可以`sudo systemctl enable bluetooth.service`
- `power on`开启蓝牙服务.
  * 如果想要开机启动`nvim /etc/bluetooth/main.conf`

  ```config
  [General]
  AutoEnable=true
  ```

- 然后使用`devices`查看可用设备, `connect MAC_address`连接即可.

## 之后使用遇到问题解决

### 1. 开始连接后等会出现连接失败

- 使用`systemctl status bluetooth.service`或`journalctl`查看关于bluetooth的错误信息, 得到如下输出:

```
bluetoothd: Unable to get connect data for Headset Voice gateway: getpeername: Transport endpoint is not connected (107)
bluetoothd: connect error: Connection refused (111)

或

Unable to get Headset Voice gateway SDP record: Host is down
```

这可能是因为MAC_address冲突(蓝牙的单耳机模式和双耳机模式切换后容易发生这种情况), 解决方法如下:

- `devices`查看添加的设备.
- `remove`移除冲突设备的MAC_address.
- 然后使用以上步骤重新连接即可.

## 参考

- [arch_Bluetooth](https://wiki.archlinux.org/index.php/Bluetooth)
- [arch_Bluetooth-headset](https://wiki.archlinux.org/index.php/Bluetooth_headset)
- [Faild to start discovery: org.bluez.Error.NotReady](https://unix.stackexchange.com/questions/508221/bluetooth-service-running-but-bluetoothctl-says-org-bluez-error-notready)
- [LinuxChina_usermod](https://linux.cn/article-10768-1.html)
