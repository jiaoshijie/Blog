---
title: "使用hugo搭建Blog记录"
date: 2019-10-01T01:31:00+08:00
draft: false
tags: ["hugo"]
slug: "using_hugo_Blog"
---

## 1. 安装[hugo](https://gohugo.io/)
  - 对于Arch系Linux系统使用 `sudo pacman -S hugo`
  - 其他安装方式去[官网](https://gohugo.io/getting-started/installing/)查看

## 2. 使用hugo创建Blog
  - `hugo new site MyBlog`
  - `MyBlog`为自己想创建的文件夹名称

## 3. 下载[hugo主题](https://themes.gohugo.io/)
  - 到项目的根目录下, 即MyBlog文件夹下
  - `git clone https://github.com/LukeSmithxyz/lugo.git themes/lugo`

  一般主题下面都有一个 *exampleSite/config.toml* 文件是这个主题的默认配置可以copy到该项目的根目录, 然后在修改一下.

## 4. 生成第一个Blog文件
  - `hugo new /posts/file_name.md`

## 5. 本地预览网站样式
  - 使用`hugo server`就可以生成预览
  - 默认是打开本地的1313端口进行预览, 使用浏览器输入`localhost:1313`打开预览

  hugo server 一些参数:

  > -D, --buildDrafts 预览时把草稿一起包括进来(默认不理睬draft: true的文件)
  > -t 指定使用的主题(或设置config.toml添加主题选项)
  > --disableLiveReload 默认hugo使用LiveReload来动态加载网页, 就是网页内容一改变就自动刷新, 可以通过该参数取消, 也可以添加的配置文件

  - 更详细的介绍使用`hugo help`或去[这里](https://gohugo.io/getting-started/usage/)

  **Notes**: 如果预览网站看不到自己创建的文件检查draft参数是否为false, 默认为true

## 6. 创建网站
  - 使用`hugo`命令创建网站
    * 该命令默认创建网站到`public`文件夹, 可以通过`publishDir`来更改默认的导出目录, 要使用**绝对路径**，相对路径好像不行.
  - `-b, --baseURL`指定hostname的根, 可以添加到配置文件
  - 主题，是否导出草稿文件，可以使用`-t, -D`来设置, 也可以通过修改配置文件来设置.
  - 使用`hugo --config="config.toml"`来指定使用哪个配置文件来生成

## 7. Development and production
  - 可以使用两个文件夹来导出网站, 分别为了Development和production.(官方推荐)
    * For Development `hugo server -wDs ~/MyBlog -d dev`
    * For production, use the default `public/` dir: `hugo -s ~/MyBlog`
  - 或者, 使用git来管理.

## 8. 将网站部署到github上
  1. 首先, 创建一个github仓库, 并命名为`username.github.io`
    * 有时, 创建的仓库*无法访问*, 可以尝试将`.io`改为`.com`试以下
  2. 将运行`hugo`命令生成的默认为`public`的文件夹初始化为git仓库, 然后添加上传
    * `git init` `git add .` `git commit -m "MyBlog"` `git remote add origin github_link` `git push -u origin master`
  3. 在浏览器中输入`username.github.io`
