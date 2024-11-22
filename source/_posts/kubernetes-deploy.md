---
title: Kubernetes 离线部署
date: 2024-11-22 15:51:19
tags:
    - Kubernetes
categories: 
    - Kubernetes
description: Kubernetes 1.21.14 离线部署
---

## 机器规划

```text
master:
    20.224.19.69    k8s-master2
    20.224.19.70    k8s-master1

nodes:
    20.224.19.68    k8s-node1
    20.224.19.65    k8s-node2
    20.224.19.66    k8s-node3
    20.201.2.9      k8s-gpu-node1
```

## 节点清理（all node）

```shell
kubeadm reset

rm -rf $HOME/.kube
rm -rf /etc/kubernetes
rm -rf /var/lib/kube*
rm -rf /var/lib/etcd/*
rm -rf /etc/cni/net.d/

iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
ipvsadm --clear
ipvsadm -C

ifconfig cni0 down
ip link delete cni0

service network restart
```

## 系统初始化检查(all node)

```shell
# 关闭防火墙
sudo systemctl stop firewalld
sudo systemctl disable firewalld
sudo systemctl status firewalld

# 关闭 selinux
sudo sed -i 's/enforcing/disabled/' /etc/selinux/config

# 禁止 swap
vim /etc/fstab

# 注释下边一行
# /dev/mapper/cl-swap swap swap defaults 0 0

# 检查 swap
free -m
```

## 配置 kubernetes（all node）

```shell
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

## 配置 docker（all node）

```shell
cat > /etc/docker/daemon.json << EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl restart docker
```

## 配置 hosts（all node）

```shell
cat >> /etc/hosts << EOF
20.224.19.69 k8s-master
20.224.19.70 k8s-master1
20.224.19.68 k8s-node1
20.224.19.65 k8s-node2
20.224.19.66 k8s-node3
20.201.2.9   k8s-gpu-node1
EOF

reboot
```

## 部署本地 yum 源（all node）

```shell
# 备份机器 yum 源
mkdir -p /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.repo* /etc/yum.repos.d/bak

# 配置机器本地源
vi /etc/yum.repos.d/local.repo

[LocalRepositry]
name=local repositry
baseurl=file:///opt/rpm_offline
gpgcheck=0
enabled=1

file:///app/dtk/kylin-v10-sp2-dtk24.04

# 创建机器 yum 源
mkdir -p /opt/rpm_offline
cd /opt/rpm_offline
cp -r /app/kubernetes_1.21.14_offline_pkg/yum_pkg/kubernetes-1.21.14/* ./
cp -r /app/kubernetes_1.21.14_offline_pkg/yum_pkg/third-party-dependency-package/* ./

createrepo -v /opt/rpm_offline
yum repolist
```

## 安装 kubelet、kubeadm、kubectl（all node）

```shell
yum -y install kubelet-1.21.14 kubectl-1.21.14 kubeadm-1.21.14
```

## 安装 haproxy

```shell
vi /etc/yum.repos.d/local.repo
  
[LocalRepositry]
name=local repositry
baseurl=file:///opt/rpm_offline_haproxy
gpgcheck=0
enabled=1
  
# 创建本地 rpm 包
mkdir -p /opt/rpm_offline_haproxy
cd /opt/rpm_offline_haproxy
cp -r /app/kubernetes_1.21.14_offline_pkg/high_availability/haproxy/rpm_pkgs/* ./
  
# 创建仓库元数据
createrepo  -v /opt/rpm_offline_haproxy
 
# 安装k8s组件
yum -y install haproxy-1.5.18
```

## 安装 keepalived

```shell
tar -zxvf keepalived-2.2.2.tar.gz
cd keepalived-2.2.2
 
./configure && make && make install
 
./configure --sysconf=/etc && make && make install
 
cp /usr/local/sbin/keepalived /usr/sbin/
```

## 导入系统依赖的 docker 镜像（all node）

```shell
docker load < /app/kubernetes_1.21.14_offline_pkg/ct_k8s_install_pkg/images_calico_3.14.2.tar.gz
docker load < /app/kubernetes_1.21.14_offline_pkg/ct_k8s_install_pkg/images.tar.gz
docker load < /app/kubernetes_1.21.14_offline_pkg/local_path_sc/busybox-latest.tar
docker load < /app/kubernetes_1.21.14_offline_pkg/local_path_sc/local-path-provisioner-master-head.tar
docker load < /app/kubernetes_1.21.14_offline_pkg/network_plugin/flannel/images_tar/flannel-cni-plugin-v1.1.2.tar
docker load < /app/kubernetes_1.21.14_offline_pkg/network_plugin/flannel/images_tar/flannel-v0.22.0.tar
```

## 配置 haproxy（all master）

```shell
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak

cat > /etc/haproxy/haproxy.cfg << EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
     
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
        
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#--------------------------------------------------------------------- 
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# kubernetes apiserver frontend which proxys to the backends
#---------------------------------------------------------------------
frontend kubernetes-apiserver
    mode                 tcp
    bind                 *:16443
    option               tcplog
    default_backend      kubernetes-apiserver   
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode        tcp
    balance     roundrobin
    server      k8s-master   20.224.19.69:6443 check
    server      k8s-master1  20.224.19.70:6443 check
#---------------------------------------------------------------------
# collection haproxy statistics message
#---------------------------------------------------------------------
listen stats
    bind                 *:1080
    stats auth           admin:awesomePassword
    stats refresh        5s
    stats realm          HAProxy\ Statistics
    stats uri            /admin?stats
EOF
```

## 配置 keepalived（all master）

```shell
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak

# master
 
cat > /etc/keepalived/keepalived.conf <<EOF
! Configuration File for keepalived
 
global_defs {
   router_id k8s
}
 
vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -60
    fall 1
    rise 2
}
 
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 250
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        20.224.19.230
    }
    track_script {
        check_haproxy
    }

}
EOF


# master1
 
cat > /etc/keepalived/keepalived.conf <<EOF
! Configuration File for keepalived
 
global_defs {
   router_id k8s
}
 
vrrp_script check_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 3
    weight -60
    fall 1
    rise 2
}
 
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 200
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        20.224.19.230
    }
    track_script {
        check_haproxy
    }
 
}
EOF

```

## 启动 keepalived 和 Haproxy

```shell 
systemctl enable keepalived && systemctl start keepalived
systemctl enable haproxy && systemctl start haproxy
```

## 验证 keepalived 和 Haproxy

```shell
# keepalived 验证：
## 查看 keepalived 服务状态：
systemctl status keepalived
 
## 在 master 节点查看网卡 eth0 是否增加了 vip 信息
ip addr

# haproxy 验证：
## 查看 Haproxy 服务状态：
systemctl status haproxy
 
## 端口检查：
netstat -lntup | grep haproxy
```

## 修改 kubeadm 配置文件

```shell
kubeadm config print init-defaults > kubeadm-init-conf.yml
```

### kubeadm 配置文件模板

```yaml
cat > /app/kubernetes_1.21.14_offline_pkg/kubeadm-init-conf.yml <<EOF
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 20.224.19.69
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
controlPlaneEndpoint: 20.224.19.230:16443
kubernetesVersion: 1.21.14
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
EOF
```


## master 节点初始化

```shell
kubeadm init --config kubeadm-init-conf.yml --upload-certs
```

## 启动 kubelet (all node)

```shell
systemctl start kubelet
systemctl enable kubelet
```

## 单 master 节点操作

```shell
kubeadm init \
--apiserver-advertise-address=20.224.19.68 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.21.14 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
```

## 加入新的 master 节点
```shell
kubeadm join 20.224.19.230:16443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:0aa42d630ec50c65363966144e624d2093d0fd3186c910a26040830a844860e2 \
        --control-plane --certificate-key 647f411544f4d11ac13e0b5ee77c9351bf47a3fcc4ef3890a8bb7f0107aeb25b
```

## node 节点操作（all node）

```shell
kubeadm join 20.224.19.230:16443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:0aa42d630ec50c65363966144e624d2093d0fd3186c910a26040830a844860e2
```

## 配置 kubectl 可执行权限（all node）

```shell
cp /app/admin.conf /etc/kubernetes/
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 为普通用户添加 kubelet 可执行权限（all node）

```shell
su root
HOME_IMSP=/app/imsp
rm -rf $HOME_IMSP/.kube
mkdir -p $HOME_IMSP/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME_IMSP/.kube/config
sudo chown -R imsp:imsp $HOME_IMSP/.kube
```

## master 节点创建网络平面（all master）

```shell
kubectl apply -f /app/kubernetes_1.21.14_offline_pkg/network_plugin/flannel/kube-flannel.yml
kubectl taint nodes --all node-role.kubernetes.io/master-
```

## 其他

```shell
# 暂停/开启 节点调度
kubectl cordon <node_name>
kubectl uncordon <node_name>

# (token过期) 重新打印token
kubeadm token create --print-join-command
kubeadm init phase upload-certs --upload-certs

# 设置 master 节点不可调度（打上污点）
kubectl taint node k8s-master node-role.kubernetes.io/master="":NoSchedule

# 去掉污点
kubectl taint node k8s-master node-role.kubernetes.io/master-node/k8s-master untainted

kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule-
kubectl taint node k8s-node1 node-role.kubernetes.io/node:NoSchedule-

kubectl taint node k8s-master node-role.kubernetes.io/master:NoSchedule+

# 查看节点信息
kubectl describe nodes node-name

# 解决普通用户执行 kubectl 命令缓慢，报 timeout 问题
chown imsp:imsp /app/imsp/.kube

# 查看安装包版本
rpm -qa | grep kube
```
