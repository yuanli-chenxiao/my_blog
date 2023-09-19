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
$ echo 123456 | passwd --stdin user_name
```

## 解锁被锁定的用户
```sh
$ pam_tally2 --user=user_name --reset
```