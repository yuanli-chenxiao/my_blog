---
title: Docker 二进制部署
date: 2024-11-22 15:28:06
tags:
    - Docker
categories: 
    - Docker
description: docker 二进制安装包部署
---

## 安装操作
docker.tgz 下载地址：https://download.docker.com/linux/static/stable/x86_64/

```shell
tar -zxvf docker-20.10.21.tgz
cp -p docker/* /usr/bin/
groupadd docker
```

## 配置文件
三个配置文件：docker.service、docker.socket、containerd.service

- docker.service
```ini
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
```

- docker.socket
```ini
[Unit]
Description=Docker Socket for the API
PartOf=docker.service
[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker
[Install]
WantedBy=sockets.target
```

- containerd.service
```ini
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target
[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999
[Install]
WantedBy=multi-user.target
```

## 拷贝配置文件到指定目录
```shell
sudo cp docker.socket /etc/systemd/system
sudo cp docker.service /etc/systemd/system
sudo cp containerd.service /etc/systemd/system
```

## 启动 Docker
```shell
systemctl enable docker.service
systemctl start docker.service
```

## 更改 Docker 镜像存储路径

默认情况下 Docker 的镜像存储位置为：/var/lib/docker，一下操作均需要 root 权限

```shell
# 查看具体位置
docker info | grep "Docker Root Dir"

# 停掉 Docker 服务
# 二者选其一
systemctl stop docker
service docker stop

# 移动整个 /var/lib/docker 目录到目的路径
mv /var/lib/docker /app/docker

# 创建软连接
ln -s /app/docker /var/lib/docker
```

## 增加普通用户执行权限

修改/var/run/docker.sock用户权限为root用户，docker组，然后将普通用户添加进docker组，这样普通用户就可以操作docker命令

```shell
# 1、添加 docker 用户组
groupadd docker

# 2、添加用户进 docker 组
gpasswd -a <user> docker

# 3、更新用户组
newgrp docker

# 4、修改 docker.sock 用户组权限
chown root:docker /var/run/docker.sock

# 5、切换普通用户，验证 docker 命令是否能执行（需要重新开启新终端） 
docker info
```
