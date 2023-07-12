---
title: Docker相关命令
date: 2023-07-10 16:44:40
tags: 
    - docker
categories: 
    - docker
description: 记录docker相关命令
---

## 启动
```shell
$ docker run -it --rm -v host_path:container_path image_name:tag /bin/bash

# -i: 以交互模式运行容器，通常与 -t 同时使用
# -t: 为容器重新分配一个伪输入终端，通常与 -i 同时使用
# --rm: 容器退出时就能够自动清理容器内部的文件系统
```

## 进入正在运行中的容器
```shell
$ docker exec -it container_id
```

## 把运行中的容器打包成镜像
```shell
$ docker commit container_id images_name:tag_no
```

## 其他
```shell
# 查看当前运行中的容器
$ docker ps -a

# 查看当前服务中的镜像
$ docker images
```

