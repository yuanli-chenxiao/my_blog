---
title: Linux命令
date: 2023-09-19 15:54:04
tags:
    - Linux
categories:
    - OS
    - Linux
description: 一些linu命令
---

## 强制修改指定用户密码
```sh
echo 123456 | passwd --stdin user_name
```

## 解锁被锁定的用户
```sh
pam_tally2 --user=user_name --reset
```

## 远程传输文件
```sh
scp file.txt remote_username@0.0.0.0:/remote/directory
```

## 查找存储时间超过30天的 .log 文件并执行删除
```sh
find /path/to/ -mtime +30 -name "*.log" -exec rm -rf {} -mtime -n +n：按照文件更改时间查找。-n指n天内；+n指n天外
```

## 更改linux源为阿里源
```sh
sudo cp /etc/apt/sources.list /etc/apt/sources-backup.list
```

复制以下配置到原文件：
```text
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```
