---
title: Python 中 docker SDK 的使用
date: 2024-11-22 15:40:24
tags:
    - Docker
categories: 
    - Python
    - Docker
description: python3 中 docker sdk 的使用
---

官网：https://docker-py.readthedocs.io/en/stable/client.html

## 安装模块
```shell
pip3 install docker -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```

## 修改 Docker 配置文件
```shell
# 登录docker所在服务器，修改docker.service文件
vim /usr/lib/systemd/system/docker.service

# 修改如下内容：
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

# 改为：
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

# 重启docker
systemctl daemon-reload
systemctl restart docker
```

## Docker SDK 常用方法

- build 构建 image
```python
import docker

# 创建 Docker 客户端
# client = docker.from_env()
client = docker.DockerClient(base_url="tcp://192.168.40.11:2375")

# 构建镜像
image, logs = client.images.build(
    path="path/to/your/dockerfile/directory",
    tag="your-image-name",
    rm=True,  # 是否在构建完成后删除中间容器
)

# 打印构建日志
for line in logs:
    print(line)
```

- push 推送 image
```python
import docker

# 创建 Docker 客户端
# client = docker.from_env()
client = docker.DockerClient(base_url='tcp://192.168.40.11:2375')

# 镜像名称
image_name = 'your-image-name'

# 登录到 Docker 镜像仓库（例如 Docker Hub）
res_dict = client.login(
    username='your-username', 
    password='your-password', 
    registry='192.168.40.11:8083'
)

# 推送镜像
for line in client.images.push(image_name, stream=True, decode=True):
    print(line)
```

- rm 删除 image
```python
import docker

# 创建 Docker 客户端
# client = docker.from_env()
client = docker.DockerClient(base_url='tcp://192.168.40.11:2375')

# 镜像名称
image_name = 'your-image-name'

# 删除镜像
try:
    client.images.remove(image_name, force=True)
    print(f"Image '{image_name}' removed successfuly.")

except docker.errors.ImageNotFound:
    print(f"Image '{image_name}' not found.")

except docker.errors.APIError as e:
    print(f"Error occurred while removing image '{image_name}': {e}")
```
