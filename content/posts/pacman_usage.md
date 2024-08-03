---
title: "pacman Usage"
date: 2024-08-04T00:29:55+08:00
tags: ["linux", "arch", "cli"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

- To use `pactree` and `paccache` install `pacman-contrib`

## To install packages

- `pacman -S package_name1 package_name2` 来安装软件
- `pacman -S $(pacman -Ssq package_regex)`来安装不太清楚名字的包
- `pacman -Sg package_group_name` 查看一个软件包组有什么软件
- `pacman -S -needed package_name1`
  - pacman 默认会重新安装软件, 即使软件已经是up to date.
  - `--needed` 会避免安装已经up to date软件
- 一些软件会有选项, 例如`Enter a selection (default=all):`
  - `1-10 15`这将安装1到10和15号软件包
  - `^5-8 ^2` 这将安装除了5-8和2号软件包之外的其他软件包

## To remove packages

- `pacman -R package_name` 只移除目标软件包, 留下其依赖的软件包
- `pacman -Rs package_name` 移除目标软件包和没有被其他软件依赖的软件包
  - `pacman -Rsu` 当移除一个group时上面的那一条命令可能拒绝执行, 尝试这一条命令
- `pacman -Rsc` 递归移除目标软件包和它依赖的全部软件及依赖它的全部软件
- `pacman -Rdd` 移除被其他包依赖的软件包, 而不移除依赖它的包. **这个命令可能会导致系统崩溃, 不推荐使用**
- `pacman -Rn package_name` 移除软件的配置文件, **但不会移除软件自己创建的, 和家目录的dot配置文件**

## To update packages

- `pacman -Syu` update full packages and system

## To get packages details

- `pacman -Ss string1 string2` 在datebase中搜索packages' name and descriptions
  - `pacman -Ss '^vim-'` 可以使用some ERE(Extend Regular Expressions)
- `pacman -Si package_name` 获取extensive information about a given package in remote
- `pacman -Qs string1 string2` 搜索本地已经安装的包的name和descriptions
  - `pacman -Q` 只搜索主动安装的包的名字
- `pacman -Qi package_name` 获取本地安装包的extensive information
- `pacman -Qo /path/to/file_name` 获取软件属于哪个软件包
- `pacman -F string1 string2` 获取远程的包的文件路径名
- `pacman -Qdt` 列出不再被依赖的软件包
- `pacman -Qet` 列出不再被依赖的软件包详细信息

## other feature

- `pacman -Sw package_name` 下载软件包不安装它
- `pacman -U /path/to/package/package_name-version.pkg.tar.zst` 安装本地软件包
- `pacman -U file:///path/to/package/package_name-version.pkg.tar.zst` 保留一个副本在pacman's cahce中
- `pacman -U http://www.example.com/repo/example.pkg.tar.zst` 安装远程软件包

## yay

- `yay -Ps` 打印系统的一些安装软件包的分析
- `yay -Yc` 清除不使用的依赖
- `yay -G aur_package_name` 下载PKGBUILD文件从ABS或AUR
- `yay -Y --gendb` 生成开发者包数据库
- `yay -Syu --devel --timeupdate` 更新系统, 检查开发者包

## paru

- `paru <target>` 交互搜索和安装target
- `paru` 和 `paru -Syu` 相同, 更新系统
- `paru -Sua` 只更新AUR packages
- `paru -Qua` 打印可用的AUR更新
- `paru -G <target>` 下载PKGBUILD文件和关联的文件
- `paru -Gp <target>` 打印target的PKGBUILD的内容
- `paru -Gc <target>` 打印target的AUR的comments
- `paru --gendb` 创建devel database

## Archlinux install deb package


### 1. 使用AUR中其他人提供的包

- using AUR(Arch Linux User Repository)

### 2. 手动安装

- 首先, 解压下载的 `*.deb` 包, `ar -x *.deb --output dirname`
- `cd dirname` 应该可以看到 `control.tar.gz` `data.tar.xz` `debian-binary`
  * `debian-binary` 显示debian包管理的信息版本(super不重要)
  * `control.tar.xz` 为软件介绍(包括依赖**非常重要**), 一些包的安全验证程序和完整性的验证程序(可以不用管)
  * `data.tar.xz` 软件安装的位置及**软件**, 可能是 `etc` `opt` `usr`
- 然后根据依赖信息安装依赖, 根据data中目录的位置将相应的软件拷贝到相应的目录.

### 3. 自己写arch's makepkg file.

- 首先, 通过`cp /usr/share/pacman/PKGBUILD.proto ./PKGBUILD`拷贝一个PKGBUILD模板.
- 然后, 设置PKGBUILD中的变量和函数的功能.
  - `pkgname`: 软件包的名称
  - `pkgver`: 软件包的版本, 通常为`数字`, `. _ -`组成
  - `pkgrel`: 必须存在, 并设置为1
  - `pkgdesc`: 软件的描述
  - `arch`: 可以运行的平台, 一般为`x86_64`
  - `url`: 下载该软件的地址
  - `license`: 发行协议
  - `depends`: 软件的依赖
  - `source`: 构建包所需要的文件组, 如果只有一个可以写一个, 也可以使很多包的地址
  - `noextract`: 一个文件组在`source`中的没有解压缩包的名称
  - `md5sums`: 包检验码, `md5sums`生成md5sums码, 如果不想使用可以复制为`SKIP`
  - `package()`: 最终安装函数
  - `prepare()`: 准备函数
  - 更多变量解释: [archwiki-PKGBUILD](https://wiki.archlinux.org/index.php/PKGBUILD)
- `pkgdir`: makepkg生成的目录默认为根目录. `srcdir`: **还不清楚**.
- 设置完成后, 在包含该PKGBUILD的目录下, 执行`makepkg`即可生成archlinux下的包.
- 使用`pacman -U pkgname`安装该软件.

### 参考

- [How to install Deb package in arch Linux](https://www.maketecheasier.com/install-deb-package-in-arch-linux/)
- [How to convert DEB packages into arch linux packages](https://www.ostechnix.com/convert-deb-packages-arch-linux-packages/)
- [Archlinux-Creating packages](https://wiki.archlinux.org/index.php/Creating_packages)
