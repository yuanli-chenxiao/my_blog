---
title: Mac删除启动台应用残留图标
date: 2023-06-15 19:50:34
tags: 
    - mac
categories: 
    - OS
    - mac
description: mac系统删除启动台应用残留图标
---

1、打开访达，点击最顶部菜单栏的“前往”-“前往文件夹”，接着输入“/private/var/folders”，然后在弹出的访达（Finder）窗口搜索栏输入：com.apple.dock.launchpad，搜索范围选择“folders”  

2、接着进入“com.apple.dock.launchpad” 文件夹里，可以看到 “db”  

3、打开终端，输入以下命令并按下空格：
```sh
$ cd /private/var/folders/hw/2j329y9n2t583q8f22kb9yw00000gp/0/com.apple.dock.launchpad/db
```
鉴于不一定每一个人都是这个路径，所以请选中“db”目录后，按下快捷键“Command + i”，查看完整路径去修改自己的命令。  

4、最后就是关键的一步，接着在“终端”输入删除遗留产物的命令行：
```sh
$ sqlite3 db "delete from apps where title='app name';"&&killall Dock
```
