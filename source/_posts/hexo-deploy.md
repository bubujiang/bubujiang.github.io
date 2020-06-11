---
title: hexo可恢复部署(github pages)
date: 2020-06-12 01:14:54
tags: 
- hexo
- deploy
categories:
- javascript
---
* 新建远程master分支,本地无需新建.
* 本地新建creator分支
* 忽略页面生成文件夹(默认为public)
* 手动推送creator分支,这样换设备就可以继续编写.
* 一键部署设置为master分支