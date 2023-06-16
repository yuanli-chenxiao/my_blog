---
title: Mac虚拟机 Parallels 18.1.1 破解
date: 2023-06-16 10:27:57
tags:
    - mac
categories: 
    - mac
    - software
description: 转载自大佬的 Parallels破解教程
---

## 18.1.1直装版，直接安装即可，无需手动破解，下载地址如下：
- 123云盘：<https://www.123pan.com/s/ZFF9-XYnPA>
- 阿里云盘：<https://www.aliyundrive.com/s/BoEeBiubLJn>

## 准备工作
#### 1、下载pd
- 官网下载地址：  
    18.1.1：<https://download.parallels.com/desktop/v18/18.1.1-53328/ParallelsDesktop-18.1.1-53328.dmg>  
    18.1.0：<https://download.parallels.com/desktop/v18/18.1.0-53311/ParallelsDesktop-18.1.0-53311.dmg>  
    18.0.1：<https://download.parallels.com/desktop/v18/18.0.1-53056/ParallelsDesktop-18.0.1-53056.dmg>

#### 2、下载破解补丁
- 18.1.1：[parallelsdesktopcrack.zip](/download/parallels/parallelsdesktopcrack_18.1.1.zip)  
- 18.1.0：[parallelsdesktopcrack.zip](/download/parallels/parallelsdesktopcrack_18.1.0.zip)  
- 18.0.1：[parallelsdesktopcrack.zip](/download/parallels/parallelsdesktopcrack_18.0.1.zip)  

## 激活方式1
- 下载补丁后解压，然后cd进入解压后的目录，然后执行 chmod +x ./install.sh && sudo ./install.sh
    ps：如果提示 Login failed: Unable to connect to Parallels Service… 忽略即可，这种一般都是已经激活成功了，可以直接去pd查看激活状态。

## 激活方式2
- 如果安装过pd17或者更早版本，可以完全卸载以确保之后能成功激活。
    ps：卸载之前请先备份好自己的虚拟机，不然卸载完就啥都没了，虚拟机文件存放目录为 ~/Parallels
- 下载pd18安装文件，安装，安装之后到激活那一步就不用继续走了，退出pd。
- 下载破解补丁到 下载目录 ，也就是 ~/Downloads, 然后解压缩，不要修改解压后的文件夹名称，这样操作都是为了保障后续执行脚本路径正确。
- 打开终端，开始执行命令破解。

    ```sh
    # 进入破解补丁目录
    cd ~/Downloads/parallelsdesktopcrack
    
    # 杀掉pd进程
    killall -9 prl_client_app
    killall -9 prl_disp_service
    
    # 复制破解补丁文件到pd目录
    sudo cp -f prl_disp_service "/Applications/Parallels Desktop.app/Contents/MacOS/Parallels Service.app/Contents/MacOS/prl_disp_service"
    sudo chown root:wheel "/Applications/Parallels Desktop.app/Contents/MacOS/Parallels Service.app/Contents/MacOS/prl_disp_service"
    sudo chmod 755 "/Applications/Parallels Desktop.app/Contents/MacOS/Parallels Service.app/Contents/MacOS/prl_disp_service"

    # 签名
    sudo codesign -f -s - --timestamp=none --all-architectures --entitlements ParallelsService.entitlements "/Applications/Parallels Desktop.app/Contents/MacOS/Parallels Service.app/Contents/MacOS/prl_disp_service"

    # 先删除原来的证书文件（如果有的话，此命令报错没关系）
    sudo rm -f /Library/Preferences/Parallels/licenses.json
    
    # 生成新的证书文件
    sudo cp -f licenses.json "/Library/Preferences/Parallels/licenses.json"
    sudo chown root:wheel "/Library/Preferences/Parallels/licenses.json"
    sudo chmod 444 "/Library/Preferences/Parallels/licenses.json"
    sudo chflags uchg "/Library/Preferences/Parallels/licenses.json"
    sudo chflags schg "/Library/Preferences/Parallels/licenses.json"
    ```

- 至此，破解完成（其他问题 [<font color="#660000">猛戳原文</font>](https://luoxx.top/archives/pd-18-active)）
