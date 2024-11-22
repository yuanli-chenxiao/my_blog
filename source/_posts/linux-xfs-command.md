---
title: Linux 磁盘相关命令
date: 2024-11-22 16:28:09
tags:
    - Linux
categories:
    - OS
    - Linux
description: Linux 磁盘相关命令 
---

```shell
# 查看磁盘空间
[root@local ~]# lsblk

[root@local ~]# pvs
  PV         VG   Fmt  Attr PSize PFree
  /dev/sdb1       lvm2 ---  4.00g 4.00g
  /dev/sdb2       lvm2 ---  4.00g 4.00g

# 创建vg
[root@local ~]# vgcreate oraclevg /dev/sdb1 /dev/sdb2
  Volume group "oraclevg" successfully created

# 卸载
[root@local ~]# vgremove oraclevg 

# vg扩容 将pv挂载到vg上
[root@local ~]# vgextend vgxx /dev/sdb2 

# 创建lv -n 指定新逻辑卷的名称 -L 指定LV大小的SIZE(M,G) （-l：小l指定LE的数量）vgname
[root@local ~]# lvcreate -n lvoracle -L 2G oraclevg
  Logical volume "lvoracle" created.

# 卸载lv
[root@local ~]# lvremove lvoracle 

# lv扩容 前提是所属vg有剩余
[root@local ~]# lvextend -L +xxG /dev/vgxx/lvxx
[root@local ~]# xfs_growfs /dev/vgxx/lvxx

# 创建文件系统
[root@local ~]# mkfs.xfs /dev/oraclevg/lvoracle

# 挂载
[root@local ~]# mount /dev/oraclevg/lvoracle /app

# 取消挂载
[root@local ~]# umount /dev/oraclevg/lvoracle 

# 取消时 对象忙， fuser 查看占用进程，然后kill
[root@local ~]# fuser -mv /app
```
