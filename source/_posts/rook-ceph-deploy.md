---
title: rook-ceph 离线部署
date: 2024-11-22 15:51:37
tags:
    - rook-ceph
categories: 
    - Kubernetes
    - rook-ceph
description: rook-ceph 离线部署
---

### 导入镜像
```bash
/app/rook-ceph/images
ls |awk '{print "docker load -i"$0}'|bash
```

### 上传harbor
#### 修改镜像名称

```bash
docker tag quay.io/ceph/ceph:v17.2.5    dockerhub.kubekey.local/rook/ceph/ceph:v17.2.5
docker tag quay.io/cephcsi/cephcsi:v3.7.2     dockerhub.kubekey.local/rook/cephcsi/cephcsi:v3.7.2
docker tag quay.io/csiaddons/k8s-sidecar:v0.5.0     dockerhub.kubekey.local/rook/csiaddons/k8s-sidecar:v0.5.0
docker tag registry.k8s.io/sig-storage/csi-attacher:v4.1.0     dockerhub.kubekey.local/rook/sig-storage/csi-attacher:v4.1.0
docker tag registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.7.0     dockerhub.kubekey.local/rook/sig-storage/csi-node-driver-registrar:v2.7.0
docker tag registry.k8s.io/sig-storage/csi-provisioner:v3.4.0     dockerhub.kubekey.local/rook/sig-storage/csi-provisioner:v3.4.0
docker tag registry.k8s.io/sig-storage/csi-resizer:v1.7.0     dockerhub.kubekey.local/rook/sig-storage/csi-resizer:v1.7.0
docker tag registry.k8s.io/sig-storage/csi-snapshotter:v6.2.1     dockerhub.kubekey.local/rook/sig-storage/csi-snapshotter:v6.2.1
docker tag rook/ceph:v1.10.10     dockerhub.kubekey.local/rook/ceph:v1.10.10
```

#### 推送镜像

```bash
docker push dockerhub.kubekey.local/rook/ceph/ceph:v17.2.5
docker push dockerhub.kubekey.local/rook/cephcsi/cephcsi:v3.7.2
docker push dockerhub.kubekey.local/rook/csiaddons/k8s-sidecar:v0.5.0
docker push dockerhub.kubekey.local/rook/sig-storage/csi-attacher:v4.1.0
docker push dockerhub.kubekey.local/rook/sig-storage/csi-node-driver-registrar:v2.7.0
docker push dockerhub.kubekey.local/rook/sig-storage/csi-provisioner:v3.4.0
docker push dockerhub.kubekey.local/rook/sig-storage/csi-resizer:v1.7.0
docker push dockerhub.kubekey.local/rook/sig-storage/csi-snapshotter:v6.2.1
docker push dockerhub.kubekey.local/rook/ceph:v1.10.10
```

#### 清理本地镜像

```shell
docker rmi dockerhub.kubekey.local/rook/cephcsi/cephcsi:v3.7.2
docker rmi dockerhub.kubekey.local/rook/csiaddons/k8s-sidecar:v0.5.0
docker rmi dockerhub.kubekey.local/rook/sig-storage/csi-attacher:v4.1.0
docker rmi dockerhub.kubekey.local/rook/sig-storage/csi-node-driver-registrar:v2.7.0
docker rmi dockerhub.kubekey.local/rook/sig-storage/csi-provisioner:v3.4.0
docker rmi dockerhub.kubekey.local/rook/sig-storage/csi-resizer:v1.7.0
docker rmi dockerhub.kubekey.local/rook/sig-storage/csi-snapshotter:v6.2.1
docker rmi dockerhub.kubekey.local/rook/ceph/ceph:v17.2.5
docker rmi dockerhub.kubekey.local/rook/ceph:v1.10.10

docker rmi quay.io/ceph/ceph:v17.2.5    
docker rmi quay.io/cephcsi/cephcsi:v3.7.2     
docker rmi quay.io/csiaddons/k8s-sidecar:v0.5.0     
docker rmi registry.k8s.io/sig-storage/csi-attacher:v4.1.0     
docker rmi registry.k8s.io/sig-storage/csi-node-driver-registrar:v2.7.0     
docker rmi registry.k8s.io/sig-storage/csi-provisioner:v3.4.0     
docker rmi registry.k8s.io/sig-storage/csi-resizer:v1.7.0     
docker rmi registry.k8s.io/sig-storage/csi-snapshotter:v6.2.1    
docker rmi rook/ceph:v1.10.10
```

### 安装

kubectl label node k8s-master app=ceph
kubectl label node k8s-master1 app=ceph

kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule-
kubectl taint node k8s-master1 node-role.kubernetes.io/master:NoSchedule-

#### 修改 operator.yaml

```txt
# 120 行
ROOK_CSI_CEPH_IMAGE: "dockerhub.kubekey.local/rook/cephcsi/cephcsi:v3.7.2"
ROOK_CSI_REGISTRAR_IMAGE: "dockerhub.kubekey.local/rook/sig-storage/csi-node-driver-registrar:v2.7.0"
ROOK_CSI_RESIZER_IMAGE: "dockerhub.kubekey.local/rook/sig-storage/csi-resizer:v1.7.0"
ROOK_CSI_PROVISIONER_IMAGE: "dockerhub.kubekey.local/rook/sig-storage/csi-provisioner:v3.4.0"
ROOK_CSI_SNAPSHOTTER_IMAGE: "dockerhub.kubekey.local/rook/sig-storage/csi-snapshotter:v6.2.1"
ROOK_CSI_ATTACHER_IMAGE: "dockerhub.kubekey.local/rook/sig-storage/csi-attacher:v4.1.0"

# 502 行
ROOK_CSIADDONS_IMAGE: "dockerhub.kubekey.local/rook/csiaddons/k8s-sidecar:v0.5.0"

# 543 行
nodeSelector:
  app: ceph

# 548 行
image: dockerhub.kubekey.local/rook/ceph:v1.10.10
```

#### 修改 cluster.yaml

```txt
# 24
image: dockerhub.kubekey.local/rook/ceph/ceph:v17.2.5

nodes:
- name: "k8s-master"
  devices:
    - name: "vdd"
- name: "k8s-master1"
  devices:
    - name: "vdd"
- name: "k8s-node1"
  devices:
    - name: "vdb"
```

#### 启动资源

```shell
kubectl create -f crds.yaml -f common.yaml -f operator.yaml

kubectl logs deploy/rook-ceph-operator -n rook-ceph
kubectl get pods -n rook-ceph -o wide
```

#### 启动集群

```shell
kubectl create -f cluster.yaml
```

#### 启动 toolbox
```shell
# 21
image: dockerhub.kubekey.local/rook/ceph/ceph:v17.2.5

kubectl create -f toolbox.yaml

# 登录
kubectl exec -it deploy/rook-ceph-tools -n rook-ceph -- /bin/bash

# 查看集群状态
ceph -s

# 查看osd状态
ceph osd df
ceph osd utilization
ceph osd pool stats
ceph osd tree

# crash
ceph crash ls
ceph crash info [id] 


# 查看ceph容量
ceph df

# 查看rados容量
rados df

# 查看pg状态
ceph pg stat

```

#### 开启 dashboard

```shell
# cluster.yaml默认为开启状态
# spec:
#   dashboard:
#     enable: true
      port：8080
      ssl: false

# 查看 dashboard 网络状态
kubectl get svc -n rook-ceph

# 修改配置 不建议使用
# type 为 NodePort
# nodePort: 30330
kubectl edit svc -n rook-ceph rook-ceph-mgr-dashboard

# 或者使用 dashboard-external-https.yaml
kubectl create -f dashboard-external-http.yaml

kubectl delete svc/rook-ceph-mgr-dashboard -n rook-ceph
```

##### dashboard账号密码

```shell
kubectl get secret rook-ceph-dashboard-password -n rook-ceph -o jsonpath="{['data']['password']}"|base64 --decode&&>echo
```

#### 创建RBD
```shell
kubectl create -f csi/rbd/storageclass.yaml
```

#### 创建 filesystem

```shell
kubectl create -f filesystem.yaml
kubectl create -f csi/cephfs/storageclass.yaml
```

#### 创建 RGW

```shell
kubectl create -f object.yaml
kubectl create -f rgw-external.yaml

kubectl -n rook-ceph get cephobjectstore
```

### 卸载
#### 清理osd
```shell
lsblk
vgs
lvs

rm -rf /var/lib/rook

sgdisk --zap-all /dev/vdb
dd if=/dev/zero of="/dev/vdb" bs=1M count=100 oflag=direct,dsync
```
