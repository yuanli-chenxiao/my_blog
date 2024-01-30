---
title: 一些关于docker的常用命令
date: 2024-01-30 23:02:32
tags: 
    - Docker
categories: 
    - Docker
description: 一些关于docker的常用命令
---

## Docker 镜像命令

### 拉取镜像
```shell
docker pull image_name:tag

-a: 是否获取仓库中所有镜像，默认为否
--disable-content-trust: 取消镜像的内容校验，默认为真
```

### 查看镜像信息
```shell
docker images

-a: 列出所有镜像文件
--digests: 列出镜像的数字摘要值，默认为否
-f: 过滤列出的镜像
```

### 添加镜像标签
```shell
docker tag image_name:tag
```

### 查看详细信息
```shell
docker inspect image_name:tag
docker history image_name:tag
```

### 搜寻镜像
```shell
docker search [option] keyword

-f: 过滤输出内容
--format string: 格式化输出内容
--limit int: 限制输出结果个数，默认为25个
--no-trunc: 不截断输出结果
```

### 存出镜像
```shell
docker save -o image_name.tar image_name:tag
```

### 载入镜像
```shell
docker load -i image_name:tag
```

### 上传镜像
```shell
docker push image_name:tag
```


## Docker 容器命令

### 新建容器
```shell
docker create -it image_name:tag
```

### 启动容器
```shell
docker start container_id
```

### 新建并启动容器
```shell
docker run -it image_name:tag /run_command

-d: 是否在后台运行
-i: 保持标准输入打开
-t: 是否分配一个伪终端
-w: 容器内的默认工作目录
-v: 挂载主机上的文件卷到容器内，host_dir:container_dir
-e: 指定容器内环境变量
--net="": 指定容器网络模式
--rm: 容器退出后是否自动删除，不能跟 -d 同时使用
--name="": 指定容器的别名
```

### 查看容器输出
```shell
docker logs container_id

-details: 打印详细信息
-f: 持续保持输出
-since string: 输出从某个时间开始的日志 
-tail string: 输出最近的若干日志
-t: 显示时间戳信息
-until string: 输出某个时间之前的日志
```

### 暂停容器
```shell
docker pause container_id
```

### 终止容器
```shell
docker stop container_id
```

### 进入容器
```shell
docker exec -it container_id /bin/bash
```

### 删除容器
```shell
docker rm container_id

-f: 是否强行终止并删除一个运行中的容器
-l: 删除容器的连接，但保留容器
-v: 删除容器挂载的数据卷
```

### 导出容器
```shell
docker export -o image_name.tar container_id
```

### 导入容器
```shell
docker import image_name.tar - image_name:tag
```