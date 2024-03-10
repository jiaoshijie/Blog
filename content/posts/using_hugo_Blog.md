---
title: "使用 hugo 搭建 Blog 记录"
date: 2019-10-01T01:31:00+08:00
draft: false
tags: ["hugo"]
slug: "using_hugo_Blog"
hidemeta: true
---

## 1. 安装 [hugo](https://gohugo.io/)

- 对于Arch系Linux系统使用 `pacman -S hugo`
- 其他安装方式去[官网](https://gohugo.io/getting-started/installing/)查看

## 2. 使用 hugo 创建 Blog

- `hugo new site <MyBlog>`
- `<MyBlog>` 为自己想创建的文件夹名称

## 3. 下载 [hugo 主题](https://themes.gohugo.io/)

- 到项目的根目录下, 即 `MyBlog` 文件夹下
- 我个人比较喜欢使用的主题为[PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- 有两种使用 git 管理主题的方式
  1. 第一种是直接将主题 clone 到指定目录
    * `git clone --depth 1 <github url> themes/<theme name>`
  2. 第二种是使用 git submodule 来管理主题(以PaperMod主题为例)
    * `git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod`
    * 然后 `cd themes/PaperMod`
    * `git ls-remote --tags` 用来查看该仓库的tag名称
    * `git fetch --tags` 将 tag 从远程仓库拉取下来
    * `git tag` 可以查看 tag 的名称
    * `git checkout <tag_name>` 切换到指定tag
    * 切换到指定 tag 后，可以主git目录使用 `git commit -a` 来更改 submodule 的 tag 进行提交
    * 日后重新 clone 该仓库到本地，不会自动 clone submodule 可以使用 `git submodule update --init --depth 1 --recursive` 来手动 clone

一般主题下面都有一个 *exampleSite/config.toml* 文件是这个主题的默认配置可以copy到该项目的根目录, 然后在修改一下.

## 4. 生成第一个Blog文件

- `hugo new /posts/<file_name>.md`
- `hugo new --kind <模板名称(没有后缀)> /posts/<file_name>.md`
  * 模板就是 `archetypes` 目录下的文件

## 5. 本地预览网站样式

- 使用 `hugo server` 就可以生成预览
- 默认是打开本地的1313端口进行预览, 使用浏览器输入 `localhost:1313` 打开预览

hugo server 一些参数:
- `-D, --buildDrafts` 预览时把草稿一起包括进来(默认不理睬 `draft: true` 的文件)
- `-t` 指定使用的主题(或设置 config.toml 添加主题选项)
- `--disableLiveReload` 默认 hugo 使用 LiveReload 来动态加载网页, 就是网页内容一改变就自动刷新, 可以通过该参数取消, 也可以添加的配置文件
- 更详细的介绍使用 `hugo help` 或去[这里](https://gohugo.io/getting-started/usage/)

**Notes**: 如果预览网站看不到自己创建的文件检查 `draft` 参数是否为 `false`，默认为 `true`

## 6. 创建网站

- 使用 `hugo` 命令创建网站
  * 该命令默认创建网站到 public 文件夹, 可以通过 `publishDir` 来更改默认的导出目录, 要使用**绝对路径**，相对路径好像不行.
- `-b, --baseURL` 指定 hostname 的根, 可以添加到配置文件
- 使用 `hugo --config="config.toml"` 来指定使用的配置文件

## 7. 将网站部署到 github 上

1. 首先, 创建一个github仓库, 并命名为 `username.github.io`
2. 将运行 hugo 命令生成的默认为 public 的文件夹初始化为 git 仓库, 然后添加上传
3. 在浏览器中输入 `username.github.io`

## 8. 更新部署的网站

将网站上传到 github 后，每次更新都是一个 commit，记录这个网站的历史没有意义。因此每次更新网站的时候可以使用以下命令来更新，或隔一段时间用一次。

1. `git checkout --orphan latest_branch` 创建一个孤儿分支
2. `git add -A`
3. `git commit -m "update website"`
4. `git branch -D main` 删除 main 分支
5. `git branch -m main` 重命名当前分支为 main 分支
6. `git push -f origin main` 强制更新到仓库
