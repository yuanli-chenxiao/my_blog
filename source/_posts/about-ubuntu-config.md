---
title: 一些关于ubuntu的设置
date: 2023-06-15 19:16:04
tags: 
    - ubuntu
categories: 
    - OS
    - ubuntu
description: 一些关于ubuntu的设置
---

## ubuntu设置静态IP

- ubuntu 18.04
```ini
$ sudo vim /etc/netplan/01-netcfg.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [1**.1**.1**.***/24]
      gateway4: 1**.1**.1**.1**
      nameservers:
              addresses: [114.114.114.114,8.8.8.8]

$ sudo netplan apply
```

- ubuntu 16.04
```ini
$ sudo vim /etc/network/interfaces

auto ens160
iface ens160 inet static
address 192.168.1.33
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 192.168.1.1

$ sudo service networking restart
$ sudo reboot

```

## Ubuntu16.04 设置时区

```shell
$ sudo timedatectl set-timezone Asia/Shanghai
```

## ubuntu16.04-安装目标软件：

```shell
$ sudo add-apt-repository ppa:jonathonf/xxx
$ sudo apt-get update
```
