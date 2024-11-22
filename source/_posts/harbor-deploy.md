---
title: Harbor 离线部署
date: 2024-11-22 15:51:47
tags:
    - Harbor
categories: 
    - Kubernetes
    - Harbor
description: Harbor 离线部署
---

## 机器规划

```text
    机器          角色          软件                                hostname
20.224.19.69    harbor1     harbor==v2.4.1、docker==v20.10.11   dockerhub.imsp
20.224.19.70    harbor2     docker-compose==v2.2.2              20.224.19.70
```

## 准备工作

- 创建外部 postgresql 数据库

```sql
CREATE USER harbor WITH PASSWORD 'Harbor12345';

# harbor_69
CREATE DATABASE registry OWNER harbor;
CREATE DATABASE notary_signer OWNER harbor;
CREATE DATABASE notary_server OWNER harbor;

# harbor_70
CREATE DATABASE registry_bak OWNER harbor;
CREATE DATABASE notary_signer_bak OWNER harbor;
CREATE DATABASE notary_server_bak OWNER harbor;

# harbor_69
ALTER DATABASE registry OWNER TO harbor;
ALTER DATABASE notary_signer OWNER TO harbor;
ALTER DATABASE notary_server OWNER TO harbor;

# harbor_70
ALTER DATABASE registry_bak OWNER TO harbor;
ALTER DATABASE notary_signer_bak OWNER TO harbor;
ALTER DATABASE notary_server_bak OWNER TO harbor;

ALTER USER harbor SUPERUSER;

GRANT SELECT,INSERT,UPDATE,DELETE ON ALL TABLES IN SCHEMA PUBLIC TO harbor;

# harbor_69
GRANT ALL PRIVILEGES ON DATABASE registry TO harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_signer TO harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_server TO harbor;

# harbor_70
GRANT ALL PRIVILEGES ON DATABASE registry_bak TO harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_signer_bak TO harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_server_bak TO harbor;
```

- 解压安装包到 /app 目录下

```shell
unzip harbor_ha.zip
```

- 创建 redis

```text
本方法没有使用外部 redis，harbor.yml 配置文件中默认配置
```

## 配置内核参数（all node）

```shell
# 在 kubernetes 节点上安装时不需要配置

sudo modprobe br_netfilter
 
sudo cat > /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
 
sudo sysctl -p
```

## 安装 docker-compose （all node）

```shell
cd /app/harbor_ha/docker-compose-v2.2.3
cp docker-compose-linux-x86_64 /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

## 安装 harbor（all node）

配置文件：此文档使用主备模式，没有配置 redis、共享目录

- hostname
- port
- data_volume (共享存储目录)（如：若使用nfs，则在3台节点上的相同目录，挂在nfs）
- 完善 external_database 的配置信息
- 完善 external_redis 的配置信息

```yaml
# Configuration file of Harbor

# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: dockerhub.imsp    # 域名或者本机 IP

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 80

# https related config
# https:
#    https port for harbor, default is 443
#  port: 443
#    The path of cert and key files for nginx
#  certificate: /your/certificate/path
#  private_key: /your/private/key/path

# Uncomment following will enable tls communication between all harbor components
# internal_tls:
#   # set enabled to true means internal tls is enabled
#   enabled: true
#   # put your cert and key files on dir
#   dir: /etc/harbor/tls/internal

# Uncomment external_url if you want to enable external proxy
# And when it enabled the hostname will no longer used
# external_url: https://reg.mydomain.com:8433

# The initial password of Harbor admin
# It only works in first time to install harbor
# Remember Change the admin password from UI after launching Harbor.
harbor_admin_password: Harbor12345

# Harbor DB configuration
# database:
#    The password for the root user of Harbor DB. Change this before any production use.
#   password: root123
#    The maximum number of connections in the idle connection pool. If it <=0, no idle connections are retained.
#   max_idle_conns: 100
#    The maximum number of open connections to the database. If it <= 0, then there is no limit on the number of open connections.
#    Note: the default number of connections is 1024 for postgres of harbor.
#   max_open_conns: 900

# The default data volume
data_volume: /app/harbor/data       # 一定要查看目标目录挂载卷大小

# Harbor Storage settings by default is using /data dir on local filesystem
# Uncomment storage_service setting If you want to using external storage
# storage_service:
# ca_bundle is the path to the custom root ca certificate, which will be injected into the truststore
# of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
#   ca_bundle:

# storage backend, default is filesystem, options include filesystem, azure, gcs, s3, swift and oss
# for more info about this configuration please refer https://docs.docker.com/registry/configuration/
#   filesystem:
#     maxthreads: 100
# set disable to true when you want to disable registry redirect
#   redirect:
#     disabled: false

# Trivy configuration
#
# Trivy DB contains vulnerability information from NVD, Red Hat, and many other upstream vulnerability databases.
# It is downloaded by Trivy from the GitHub release page https://github.com/aquasecurity/trivy-db/releases and cached
# in the local file system. In addition, the database contains the update timestamp so Trivy can detect whether it
# should download a newer version from the Internet or use the cached one. Currently, the database is updated every
# 12 hours and published as a new release to GitHub.
trivy:
  # ignoreUnfixed The flag to display only fixed vulnerabilities
  ignore_unfixed: false
  # skipUpdate The flag to enable or disable Trivy DB downloads from GitHub
  #
  # You might want to enable this flag in test or CI/CD environments to avoid GitHub rate limiting issues.
  # If the flag is enabled you have to download the `trivy-offline.tar.gz` archive manually, extract `trivy.db` and
  # `metadata.json` files and mount them in the `/home/scanner/.cache/trivy/db` path.
  skip_update: false
  #
  # insecure The flag to skip verifying registry certificate
  insecure: false
  # github_token The GitHub access token to download Trivy DB
  #
  # Anonymous downloads from GitHub are subject to the limit of 60 requests per hour. Normally such rate limit is enough
  # for production operations. If, for any reason, it's not enough, you could increase the rate limit to 5000
  # requests per hour by specifying the GitHub access token. For more details on GitHub rate limiting please consult
  # https://developer.github.com/v3/#rate-limiting
  #
  # You can create a GitHub token by following the instructions in
  # https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line
  #
  # github_token: xxx

jobservice:
  # Maximum number of job workers in job service
  max_job_workers: 10

notification:
  # Maximum retry count for webhook job
  webhook_job_max_retry: 10

chart:
  # Change the value of absolute_url to enabled can enable absolute url in chart
  absolute_url: disabled

# Log configurations
log:
  # options are debug, info, warning, error, fatal
  level: info
  # configs for logs in local storage
  local:
    # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
    rotate_count: 50
    # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
    # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
    # are all valid.
    rotate_size: 200M
    # The directory on your host that store log
    location: /var/log/harbor

  # Uncomment following lines to enable external syslog endpoint.
  # external_endpoint:
  #   # protocol used to transmit log to external endpoint, options is tcp or udp
  #   protocol: tcp
  #   # The host of external endpoint
  #   host: localhost
  #   # Port of external endpoint
  #   port: 5140

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version: 2.4.0

# Uncomment external_database if using external database.
# 69 连接主数据库，70 连接备数据库 （_bak后缀）
external_database:
  harbor:
    host: 20.12.6.122
    port: 5432
    db_name: registry
    username: harbor
    password: Harbor12345
    ssl_mode: disable
    max_idle_conns: 2
    max_open_conns: 0
  notary_signer:
    host: 20.12.6.122
    port: 5432
    db_name: notary_signer
    username: harbor
    password: Harbor12345
    ssl_mode: disable
  notary_server:
    host: 20.12.6.122
    port: 5432
    db_name: notary_server
    username: harbor
    password: Harbor12345
    ssl_mode: disable

# Uncomment external_redis if using external Redis server
# external_redis:
#   # support redis, redis+sentinel
#   host for redis: :Imsp@2024@20.12.6.117:7000
#   host for redis+sentinel:
#   # <host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>
#   host: 20.12.6.117:7000
#   password: Imsp#2024
#   sentinel_master_set must be set to support redis+sentinel
#   sentinel_master_set:
#     db_index 0 is for core, it's unchangeable
#   registry_db_index: 1
#   jobservice_db_index: 4
#   chartmuseum_db_index: 3
#   trivy_db_index: 5
#   idle_timeout_seconds: 60

# Uncomment uaa for trusting the certificate of uaa instance that is hosted via self-signed cert.
# uaa:
#   ca_file: /path/to/ca

# Global proxy
# Config http proxy for components, e.g. http://my.proxy.com:3128
# Components doesn't need to connect to each others via http proxy.
# Remove component from `components` array if want disable proxy
# for it. If you want use proxy for replication, MUST enable proxy
# for core and jobservice, and set `http_proxy` and `https_proxy`.
# Add domain to the `no_proxy` field, when you want disable proxy
# for some special registry.
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

# metric:
#   enabled: false
#   port: 9090
#   path: /metrics

# Trace related config
# only can enable one trace provider(jaeger or otel) at the same time,
# and when using jaeger as provider, can only enable it with agent mode or collector mode.
# if using jaeger collector mode, uncomment endpoint and uncomment username, password if needed
# if using jaeger agetn mode uncomment agent_host and agent_port
# trace:
#   enabled: true
#     set sample_rate to 1 if you wanna sampling 100% of trace data; set 0.5 if you wanna sampling 50% of trace data, and so forth
#   sample_rate: 1
#     # namespace used to differenciate different harbor services
#     namespace:
#     attributes is a key value dict contains user defined attributes used to initialize trace provider
#   attributes:
#     application: harbor
#   # jaeger should be 1.26 or newer.
#   jaeger:
#     endpoint: http://hostname:14268/api/traces
#     username:
#     password:
#     agent_host: hostname
#     # export trace data by jaeger.thrift in compact mode
#     agent_port: 6831
#   otel:
#     endpoint: hostname:4318
#     url_path: /v1/traces
#     compression: false
#     insecure: true
#     timeout: 10s
```

## 执行安装（all node）

```shell
# 加载harbor依赖docker镜像
cd /app/harbor_ha/harbor
python docker_load.py
 
# 解压安装包
cd /app/harbor_ha/harbor
tar -zxvf ./harbor-offline-installer-v2.4.1.tgz
 
# 配置文件
cd /app/harbor_ha/harbor/harbor
cp harbor.yml.tmpl harbor.yml
vim harbor.yml     # 将上一小节配置文件内容加入 harbor.yml （前提：根据自身环境更新配置文件）
 
# 创建本地 data 目录 (可选：测试时可用；生产使用分布式存储时，可不必创建)
mkdir -p /app/harbor/data
 
# 将 harbor.yml 配置文件的内容注入到各组件的配置文件中
cd /app/harbor_ha/harbor/harbor
./prepare
 
# 启动 harbor
cd /app/harbor_ha/harbor/harbor
./install.sh
```

"insecure-registries": ["dockerhub.imsp", "20.224.19.70:80"]

## 验证
都为 healthy 为正常

```shell
docker ps | grep harbor

[imsp@k8s-master ~]$ docker ps | grep harbor
3be8de9029fe   goharbor/nginx-photon:v2.4.1                          "nginx -g 'daemon of…"   16 hours ago     Up 16 hours (healthy)   0.0.0.0:80->8080/tcp, :::80->8080/tcp   nginx
15b6c417d494   goharbor/harbor-core:v2.4.1                           "/harbor/entrypoint.…"   16 hours ago     Up 16 hours (healthy)                                           harbor-core
0eac96e992fd   goharbor/harbor-registryctl:v2.4.1                    "/home/harbor/start.…"   16 hours ago     Up 16 hours (healthy)                                           registryctl
100feeff07f1   goharbor/harbor-portal:v2.4.1                         "nginx -g 'daemon of…"   16 hours ago     Up 16 hours (healthy)                                           harbor-portal
9f33de87f2e3   goharbor/registry-photon:v2.4.1                       "/home/harbor/entryp…"   16 hours ago     Up 16 hours (healthy)                                           registry
8dec0eee1eda   goharbor/redis-photon:v2.4.1                          "redis-server /etc/r…"   16 hours ago     Up 16 hours (healthy)                                           redis
d785c4f67f73   goharbor/harbor-log:v2.4.1                            "/bin/sh -c /usr/loc…"   16 hours ago     Up 16 hours (healthy)   127.0.0.1:1514->10514/tcp               harbor-log
```

## 调试
通过 docker-compose 启停 harbor：

```shell
# 需进入到 docker-compose.yml 所在目录
cd /app/harbor_ha/harbor/harbor
 
# 启动 harbor
docker-compose up -d
 
# 停止 harbor
docker-compose down
 
# 重启 harbor
docker-compose restart
```
## 生成 harbor-secret
```shell
kubectl -n <ns> create secret docker-registry harbor-secret --docker-username=admin --docker-password=Harbor12345 --docker-server=20.224.19.69
```
