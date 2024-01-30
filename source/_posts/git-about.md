---
title: git常用操作
date: 2023-09-19 17:32:35
tags:
    - Git
categories: 
    - Git
description: git常用命令
---

## 解决git老要输入账户密码问题
```sh
git config --global credential.helper store
```

## git压缩命令
```sh
git archive --format=zip --output=/path/to/object_name.zip branch_name
```

## 配置git代理
```sh
git config --global http.proxy 127.0.0.1:7890
git config --global https.proxy 127.0.0.1:7890
```
