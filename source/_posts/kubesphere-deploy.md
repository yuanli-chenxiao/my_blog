---
title: Kubesphere 离线部署
date: 2024-11-22 15:51:27
tags:
    - Kubesphere
categories: 
    - Kubernetes
    - Kubesphere
description: Kubesphere 离线部署
---

## 准备工作

```text
Kubernetes: v1.21.14
Kubesphere: 3.3.2
Docker: 20.10.11
Harbor: 2.4.1
```

```text
# 目录文件

kubesphere-installer.yaml 
cluster-configuration.yaml 

kubesphere-delete.sh    # 彻底卸载 kubesphere 脚本
offline-installation-tool.sh    # 镜像操作脚本

images/
    image_tar.txt   # 镜像 tar 包清单
    local_images.txt    # 实际部署镜像清单
    kubesphere-images.txt   # 官方提供镜像清单
```

```shell
# 检查集群中是否有默认 StorageClass (准备默认 StorageClass 是安装 Kubesphere 的前提条件)。
# 本部署方式选择 rook-ceph-block

# 查看集群中的 StorageClass
kubectl get sc

# 设置默认 StorageClass

kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## 安装步骤

- 编辑 cluster-configuration.yaml

    ```yaml
    # 15 行（修改为实际 harbor 域名）
    local_registry: dockerhub.imsp
    ```

- 编辑 kubesphere-installer.yaml

    ```yaml
    # 290 行
    image: dockerhub.imsp/kubesphereio/ks-installer:v3.3.2
    ```

- 执行安装

    ```shell
    kubectl apply -f kubesphere-installer.yaml
    kubectl apply -f cluster-configuration.yaml
    ```

## 验证安装

```shell
kubectl get pod -A | grep kubes

# kubesphere-system              ks-installer-6f6d4f648-fzj4f                            1/1     Running     0          133m

kubectl logs pod/ks-installer-6f6d4f648-fzj4f -n kubesphere-system

Collecting installation results ...
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://20.224.19.69:30880
Account: admin
Password: P@88w0rd
NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2024-08-29 15:11:21
#####################################################
```
