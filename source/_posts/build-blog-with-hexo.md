---
title: GithubPages+hexo搭建博客
date: 2019-03-12 17:09:00
categories: 
- 其他
tags:
- hexo
---

记录如何在OS X上搭建Hexo博客并部署到Githubpages。

<!-- more -->

hexo的[中文官方指南](https://hexo.io/zh-cn/docs/setup.html)在这里。


# 准备
 
 你要有个Github账号和一个Gitpage仓库。
 
# 安装：
## 安装Node.js & Git
 
 
 https://nodejs.org/en/
 
 https://git-scm.com/
 
 有博客说hexo编译会依赖Xcode，Xcode在App Store下载即可。
 
## 安装Hexo

```bash
 sudo npm install -g hexo-cli
```

## 初始化Hexo

 在你想要作为博客目录的目录下，打开终端：
 
```bash
 hexo init
 npm install
```

## 安装hexo server

```bash
 sudo npm install hexo-server
```

 到这里hexo生成页面的所有组件就安装完啦！
 可以先看看默认的页面什么样：
 
```bash
 hexo g (/generate)
 hexo s (/server)
```
 看到 Hexo is running at http://localhost:4000/. 
 就可以看看默认的页面啦。
 
#基本配置
## 关联Github

 在Hexo安装目录里找到_config.yml文件，打开，修改最后面的deploy段为：
 
```
 deploy: 
  type: git 
  repo: https://github.com/你的用户名/你的用户名.github.io.git 
  branch: master
```
  然后 hexo d (/deploy)
  就部署到你的gitpage啦。
  
# hexo基本操作

写文章

```
hexo new "postName"
```
生成静态页面

```
hexo g
```
本地预览

```
hexo s
```
部署

```
hexo d
```
这里有一个可能忘记的点：在部署之前需要安装hexo-deployer-git。

```
npm install --save hexo-deployer-git
```
清除缓存

```
hexo clean
```	
详细参见 [指令 | hexo](https://hexo.io/zh-cn/docs/commands.html)。 

 
 
# 个性化


 在[主题列表](https://hexo.io/themes/)上找到心仪主题，clone到博客目录下的themes目录，再更改_config.yml文件的theme参数为想更换的主题名称即可。注意此处的主题名称是存放主题的文件夹的名称。
 
 参考了这两篇博客：
 
 [hexo的next主题个性化配置教程](https://segmentfault.com/a/1190000009544924)
 
 [HEXO-NEXT主题个性化配置](http://mashirosorata.vicp.io/HEXO-NEXT%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE.html)
 
 

# 源码自动备份

 参考这两篇博客。
 
 [备份Hexo博客源文件](https://notes.doublemine.me/2015-04-06-%E5%A4%87%E4%BB%BDHexo%E5%8D%9A%E5%AE%A2%E6%BA%90%E6%96%87%E4%BB%B6.html)
 
 [自动备份Hexo博客源文件](https://notes.doublemine.me/2015-07-06-%E8%87%AA%E5%8A%A8%E5%A4%87%E4%BB%BDHexo%E5%8D%9A%E5%AE%A2%E6%BA%90%E6%96%87%E4%BB%B6.html)
 
