---
title: hexo博客迁移记录
date: 2019-7-28
tags: hello
categories: Testing
---

本文并不是从头开始搭建hexo博客的教程，而是因为最近个人重装了电脑系统，需求迁移原来搭建好的hexo博客，为了防止将来还需要迁移，特记录下操作流程，以便将来查看。

迁移博客的工作量要比从头开始搭建简单很多，很多插件的服务端配置都不需要重新设置，只需要在本地做相应的操作即可。

<!--more-->
### 安装hexo

第一步当然是安装相应的软件和配置好环境。需要安装的软件有Node.js和Git,去官网下载安装即可。

```powershell
git -v
node -v
hexo -v
```

当Node.js和Git都安装好后就可以正式安装Hexo了，终端执行如下命令：
```powershell
sudo npm install -g hexo
```
### git更换源
```powershell
npm config set registry https://registry.npm.taobao.org 
npm info underscore (输出正常反馈信息则说明换源成功)
```
### hexo部署插件
```powershell
本地测试的时候需要用hexo server
npm i hexo-server
将文章部署到github上的模块
npm install hexo-deployer-git --save
安装RSS插件
npm install hexo-generator-feed --save
添加Sitemap,加速网页收录速度
npm install hexo-generator-sitemap --save
```

.deploy：执行hexo deploy命令部署到GitHub上的内容目录 
public：执行hexo generate命令，输出的静态网页内容目录 
scaffolds：layout模板文件目录，其中的md文件可以添加编辑 
scripts：扩展脚本目录，这里可以自定义一些javascript脚本 
source：文章源码目录，该目录下的markdown和html文件均会被hexo处理。该页面对应repo的根目录，404文件、favicon.ico文件，CNAME文件等都应该放这里，该目录下可新建页面目录。 
_drafts：草稿文章 
_posts：发布文章 
themes：主题文件目录 
_config.yml：全局配置文件，大多数的设置都在这里 
package.json：应用程序数据，指明hexo的版本等信息，类似于一般软件中的关于按钮

Hexo原理就是hexo在执行hexo generate时会在本地先把博客生成的一套静态站点放到public文件夹中，在执行hexo deploy时将其复制到.deploy文件夹中。Github的版本库通常建议同时附上README.md说明文件，但是hexo默认情况下会把所有md文件解析成html文件，所以即使在线生成了README .md，它也会在你下一次部署时被删去。怎么解决呢？ 
在执行hexo deploy前把在本地写好的README.md文件复制到.deploy文件夹中，再去执行hexo deploy。
### hexo博客更新博文
```powershell
hexo g       # 初始化文件
hexo s       # 本地部署
hexo d       # 发布到github
```
先记录到这，一会出现问题都会在这里更新。


