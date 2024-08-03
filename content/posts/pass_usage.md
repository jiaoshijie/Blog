---
title: "Pass Usage"
date: 2024-08-03T23:06:40+08:00
tags: ["linux", "cli"]
categories: ""
author: "Jiao shijie"
draft: false
hidemeta: true
math: false
---

## install pass

- `pacman -S pass`

## 设置GnuPG

pass使用PGP(Pretty Good Privacy)密钥, 最常用的PGP实现是GnuPG(GPG), 它随linux系统一起安装.

- `gpg --generate-key` 来产生PGP密钥
  * 在这个过程中会提示你输入你的名字和电子邮箱
  * 并为密钥创建一个密码
  * 密钥加密, 密码解密(但只有密码没有密钥也无法解密)
- `gpg --list-keys` 可以查看你创建的PGP密钥

```bash
$ gpg --list-keys
/home/seth/.gnupg/pubring.kbx
-----------------------------
pub  ed25519 2022-01-06 [SC] [expires: 2024-01-06]
    2BFF94286461216C907CBA52F067996F13EF10D8
uid  [ultimate] Seth Kenlon <seth@example.com>
sub  cv25519 2022-01-06 [E] [expires: 2024-01-06]
```

### export and import gpg key

- export a gpg public key `gpg --export [-a] {key-id} > public-key.asc`
- export a gpg private key `gpg --export-secret-keys [-a] {key-id} > private-key.asc`
- import a gpg key `gpg --import [public|private key]`
- backup a gpg public key `gpg --export --export-options backup --output public.gpg`
- backup a gpg private key `gpg --export-secret-keys --export-options backup --output private.gpg`
- [export gpg private key and public key file](https://itslinuxfoss.com/export-gpg-private-key-and-public-key-file/)

### change expiration data of a gpg key

- `gpg --edit-key [key-id]`
  + `> expire` subcommand
- [change expiration date gpg key](https://www.g-loaded.eu/2010/11/01/change-expiration-date-gpg-key/)

## 设置pass

- `pass init 2BFF94286461216C907CBA52F067996F13EF10D8` 初始pass仓库

## 存储密码

- `pass add {www.example.com}`
- `pass generate {www.example.com} 20 -c` 自动为www.example.com网站创建一个20位随机密码并传送到系统剪切板

## 编辑密码

- `pass edit {www.example.com}` 可以添加一些信息

```
bd%dc$3a49af49498bb6f31bc964718C
user: seth123
url: example.com
```

## 获取密码

- `pass show {www.example.com}`
- `passmenu`

## 查找密码

- `pass grep example`

```bash
$ pass grep example
www.example.com:
url: www.example.org
```

## ref

- [Manage your passwords in the Linux terminal](https://opensource.com/article/22/1/manage-passwords-linux-terminal)
