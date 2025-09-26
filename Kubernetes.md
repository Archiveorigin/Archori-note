# Containerd

## Base Environment

**基本包**

```bash
yum install -y device-mapper-persistent-data lvm2 wget net-tools nfs-utils lrzsz gcc gcc-c++ make cmake libxml2-devel openssl-devel curl curl-devel unzip sudo ntp libaio-devel wget vim ncurses-devel autoconf automake zlib-devel  python-devel epel-release openssh-server socat  ipvsadm conntrack telnet ipvsadm

device-mapper-persistent-data：设备映射器持久数据，用于设备映射器的持久化存储。
lvm2：逻辑卷管理器，用于管理逻辑卷。
wget：用于从网络下载文件的命令行工具。
net-tools：包含了一系列网络工具，如ifconfig和netstat，用于配置和管理网络。
nfs-utils：NFS(Network File System)工具，用于搭建和管理NFS网络文件系统。
lrzsz：提供了用于在UNIX系统和计算机终端之间传输文件的工具。
gcc：GNU编译器集合，用于编译C语言和C++程序。
gcc-c++：GNU编译器集合的C++编译器。
make：用于自动化编译和安装程序的工具。
cmake：用于跨平台软件构建的工具。
libxml2-devel：libxml2的开发库，用于开发基于XML的应用程序。
openssl-devel：OpenSSL的开发库，用于在应用程序中使用加密和安全功能。
curl：用于从命令行或脚本中进行URL数据传输的工具。
curl-devel：libcurl的开发库，用于在应用程序中使用libcurl进行URL数据传输。
unzip：用于解压缩ZIP文件的命令行工具。
sudo：用于以其他用户的身份执行命令的工具，通常用于提升权限。
ntp：网络时间协议客户端，用于同步系统时钟。
libaio-devel：异步I/O（AIO）的开发库，用于开发异步I/O应用程序。
vim：文本编辑器，通常用于在命令行中编辑文本文件。
ncurses-devel：ncurses的开发库，用于在文本终端上显示复杂的图形界面。
autoconf：用于自动配置软件包的工具。
automake：用于自动生成Makefile文件的工具。
zlib-devel：zlib的开发库，用于在应用程序中进行数据压缩和解压缩。
python-devel：Python的开发库，用于在应用程序中使用Python编程语言。
epel-release：Extra Packages for Enterprise Linux (EPEL)软件源的发布包，用于安装额外的软件包。
openssh-server：OpenSSH服务器，用于远程访问和管理服务器。
socat：多功能的网络工具，用于在不同类型的网络连接之间传输数据。
ipvsadm：IPVS管理工具，用于配置Linux内核中的IPVS（IP Virtual Server）负载均衡。
conntrack：用于查看和管理Linux内核连接跟踪表的工具。
telnet：用于通过Telnet协议连接到远程主机的工具。
```

**关闭防火墙和SELinux**

```bash
systemctl stop firewalld
setenforce 0
```

**配置时间同步并创建计划任务**

```bash
yum -y install chrony
systemctl enable chronyd --now
vim /etc/chrony.conf
server ntp1.aliyun.com iburst
server ntp2.aliyun.com iburst
server ntp1.tencent.com iburst
server ntp2.tencent.com iburst

crontab -e
* * * * * /usr/bin/systemctl restart crond
```

**安装Docker服务**

```bash
# 安装docker阿里云在线源
yum install yum-utills -y
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum clean all && yum makecache

# 或者直接手写repo
[docker-ce-stable]
name=Docker CE Stable - CentOS
baseurl=http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

# 或者直接wget
sudo wget -O /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

**修改内核参数**

```bash
modprobe br_netfilter
vim /etc/sysctl.d/docker.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

sysctl -p /etc/sysctl.d/docker.conf

net.bridge.bridge-nf-call-ip6tables = 1：
这个参数启用了 IPv6 的iptables netfilter hook，允许 Linux 内核在网络层面上进行 IPv6 数据包的过滤和处理。在 Kubernetes 中，这个参数通常需要启用，因为 Kubernetes 网络通常是基于 iptables 实现的，而且在一些网络插件中可能会使用到 IPv6。这个参数的启用确保了 Linux 内核在处理 IPv6 数据包时会经过 iptables 过滤，从而确保网络功能的正常运行。

net.bridge.bridge-nf-call-iptables = 1：
这个参数启用了 IPv4 的iptables netfilter hook，类似于前一个参数，但是针对 IPv4 数据包。它允许 Linux 内核在网络层面上进行 IPv4 数据包的过滤和处理。在 Kubernetes 中，这个参数通常需要启用，因为 Kubernetes 网络通常是基于 iptables 实现的，而且在一些网络插件中可能会使用到 Ipv4。这个参数的启用确保了 Linux 内核在处理 Ipv4 数据包时会经过 iptables 过滤，从而确保网络功能的正常运行。

net.ipv4.ip_forward = 1：
这个参数启用了 Linux 内核的 IP 转发功能，允许 Linux 主机将收到的数据包从一个网络接口转发到另一个网络接口。在 Kubernetes 中，Pod 可能会跨越多个节点进行通信。例如，当一个 Pod 需要访问另一个 Pod 或外部服务时，网络流量可能需要通过不同的节点进行路由。启用 IP 转发功能允许 Linux 主机将收到的数据包从一个网络接口转发到另一个网络接口，从而实现跨节点通信。
```

**安装docker**

```bash
yum install -y docker-ce-24.0.6
systemctl start docker
systemctl enable docker
ststemctl status docker -l
```

**配置docker镜像加速器**

```bash
# 登陆阿里云镜像仓库
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

vim /etc/docker/daemon.json
{
 "registry-mirrors":["https://cfhhdrrf.mirror.aliyuncs.com","https://docker.lmirror.top","https://docker.m.daocloud.io", "https://hub.uuuadc.top","https://docker.anyhub.us.kg","https://dockerhub.jobcher.com","https://dockerhub.icu","https://docker.ckyl.me","https://docker.awsl9527.cn","https://docker.laoex.link"],
"insecure-registries":["192.168.40.62","harbor"]
}

systemctl daemon-reload
systemctl restart docker
docker pull centos
```

## Base Command

```bash
# 查找镜像
docker search image_name
# 拉取镜像
docker pull image_name
# 查看本地镜像
docker images
# 加载离线文件里的镜像
docker load -i file_name
# 制作镜像为离线文件
docker save -o file_name image_name
# 删除镜像
docker rmi -f images_name/images_id
# 创建并进入容器
docker rum --name=podname -it images_name /bin/bash
# 查看所有容器
docker ps -a
# 以守护进程(后台)的方式运行容器
docker rum --name=podname -dt images_name
# 进入运行状态的容器
docker exec -it pod_name /bin/bash
# 停止运行的容器
docker stop pod_name


```

**docker run**

```bash
docker run [OPTIONS] IMAGE [COMMAND] [AEG..]

IMAGE 指定用于创建容器的镜像名称 如nginx
COMMAND 容器启动时要执行的命令
ARG 传递给COMMAND的参数

# --name 指定容器名称 缺省则默认生成随机名称
docker run --name my-container nginx

# --hostname 指定容器的主机名
docker run --hostname my-hostname nginx

# --rm 容器退出后自动删除 适合临时容器
docker run --rm nginx

# -d / --detach 守护模式
docker run -d nginx

# -it 交互模式运行容器
docker run -it ubuntu /bin/bash

# --restart 容器的重启策略 no 默认值不重启 always 退出自动重启 on-failure 仅在错误退出时重启 unless-stopped 始终重启 除非手动停止
docker run --restart=always nginx

# -p / --publish 端口映射 格式为 主机端口:容器端口
docker run -p 8080:80 nginx

# -P / --publish-all 随机映射容器的所有暴露端口到主机端口
docker run -P nginx

# -v / --volume 挂载卷到容器 格式为 主机路径:容器路径
docker run -v /data:/app/data nginx

# --mount 高级挂载选项 可以指定更多参数 如类型 读写权限等
docker run --mount type=bind,source=/data,target=/app/data nginx

# --network 指定容器使用网络类型 默认bridge
docker run --network=host nginx

# --dns 配置容器使用的DNS服务器
docker run --dns=8.8.8.8 nginx

# --cpus 限制容器使用的cpu数量
docker run --cpus="2.5" nginx

# --memory 限制容器内存使用
docker rum --memory="512m" nginx

# --memory-swap 指定最大内存和交换分区总量
docker run --memory="512m" --memmory-swap="1g" nginx

# -e / -env 设置环境变量
docker run -e ENV_VAR=value nginx

# --env-file 从文件中加载环境变量
docker run --env-file ./env.list nginx

# --log-driver 指定日志驱动程序 如json-file或syslog
docker run --log-deiver=syslog nginx

# --log-opt 配置日志驱动程序的选项
docker run --log-driver=syslog --log-opt tag="{{.Name}}" nginx

# -u / --user 指定运行容器的用户
docker run -u 1000:1000 nginx

# --privileged 授予容器更高权限 允许容器访问主机设备
docker run --privileged nginx

# --health-cmd 设置健康检查命令
docker run --health-cmd="curl -f http://localhost/ || exit 1" nginx

# --health-interval 设置健康检查时间间隔
docker run --health-interval=30s nginx

# --detach-keys 设置退出容器时的键组合 默认Ctrl+P Ctrl+Q
docker run --detach-keys="ctrl-e,e" nginx

# --entryporint 重写镜像的默认入口点
docker run --entrypoint "/bin/bash" nginx
```

**docker exec**

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG..]

CONTAINER 目标容器的名称或ID
COMMAND 要在容器中执行的命令
ARG 传递给COMMAND的参数

# -i / --interactive 以交互式运行命令 保持输入流打开
docker exec -i my_container bash

# -t / --tty 分配一个伪终端 通常与-i配合使用 进入交互式终端环境
docker exec -it my_container bash

# -u / --user 指定在容器中运行命令的用户 格式为 用户名 或 UID:GID
docker exec -u root my-container ls /root

# --env / -e 设置运行命令时的环境变量
docker exec -e MY_VAR=value my-container printenv MY_VAR

# --env-file 从文件中加载环境变量
docker exec --env-file ./env.list my-container env

# -w / --workdir 指定在容器内执行命令时的工作目录
docker exec -w /app my-container ls

# --detach 在分离模式下运行命令 不等待命令执行完成
docker exec --datach my-container some-long-task
```

## Build Dockerfile

Dockerfile 是用来定义 Docker 镜像构建过程的文本文件 他包含了一系列的指令和参数 告诉 Docker 如何组装一个镜像

### Dockerfile 基本结构

1. 基础镜像指令 （FROM）

   指定基于哪个已有镜像构建新的镜像 每个dockerfile必须以FROM开始

   示例 FROM ubuntu:20.04

   

2. 工作目录设置 （WORKDIR,可选）

   设置容器内部工作目录

   示例 WORKDIR /app

   

3. 复制文件 （COPY / ADD / 可选）

   将文件或目录从构建上下文复制到镜像中指定路径

   示例 COPY . .

   ADD 可以把压缩包解压 把解压后的目录拷贝到镜像

   

4. 运行命令 （RUN / 可选）

   在镜像构建过程中执行的命令 可以安装软件包等

   示例 RUN apt-get update && apt-get install -y nginx

   

5. 容器启动时执行命令 （CMD / ENTRYPOINT / 可选）

   容器启动后执行的默认命令 类似开启容器自动运行的命令

   示例 CMD ["nginx","-g","daemon off;"]

   

6. 暴露端口 （EXPOSE / 可选）

   声明容器运行时的服务将监听的端口

   示例 EXPOSE 80

### 使用Dockerfile构建容器

基础命令：`docker build`

```bash
docker build [OPTIONS] PATH | URL | -
PATH 构建上下文的路径（通常是Dockerfile所在目录）
URL Dockerfile的URL地址
- 从标准输入读取Dockerfile

# -t / --tag 给构建的镜像指定标签 格式 name:tag
docker build -t my_image:1.0 .

# --file / -f 指定Dockerfile文件的路径
docker build -f ./custom.Dockerfile -t my-image .

# . 构建上下文 通常使用当前目录作为构建上下文 Docker引擎会将指定路径下的所有文件发送给Docker守护进程
docker build -t my-image .

# --built-context 允许为构建中的特定上下文指定不同路径或URL
docker build --build-context app=./app=context -t my-image .

# --no-cache 不使用缓存构建镜像 强制重新构建每一层
docker build --no-cache -t my-image .

# --pull 始终尝试从远程仓库拉取最新的基础镜像
docker build --pull -t my-image .

# --target 指定Dockerfile中的构建目标 仅构建到指定的阶段
docker build --target builder -t my-builder .

# --build-arg 传递构建时的参数 用于Dockerfile中的ARG
docker build --build-arg ENV_VAR=value -t my-image .
```



## Data Holding

在 Docker 中 容器中的文件系统都是临时的 如果容器被删除或重新创建 未保存的数据就会丢失

### Bind Mounts

将宿主机上的特定目录挂载到容器内

```bash
# 使用-v参数将主机目录挂载至容器中

docker run -d -p 8080:80 -v /docker/nginx/index.html:/var/www/html nginx:v1 --name my_container_for_nginx

```



### Volumes

由 Docker 管理的特殊目录 用于存储数据

```bash
# 使用docker命令创建volume 此volume只由docker管理 独立于宿主机的文件系统 实现方式更加严谨 适合生产环境使用

docker volume create myvolume
docker run -d -p 8080:80 --name my_container -v myvolume:/usr/share/nginx/html nginx:v1
[Kubernetes Lab-1 nginx] docker volume inspect myvolume
[
    {
        "CreatedAt": "2025-01-18T11:21:56+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": null,
        "Scope": "local"
    }
]
[Kubernetes Lab-1 nginx] cd /var/lib/docker/volumes/myvolume/_data
[Kubernetes Lab-1 _data] ls
404.html  50x.html  index.html  nginx-logo.png  poweredby.png

```

## Virtual Network

### 虚拟网桥

`docker0`

```bash
# 虚拟网桥是宿主机和容器之间的一个虚拟交换机 负责将宿主机的流量转发到docker的虚拟网卡中
```

### veth设备

```bash
# veth 设备成对出现 用于联通宿主机和容器 是容器的虚拟网卡 流量经过docker0网桥转发进veth设备 再传入容器中

```

### 网络模式

1. Bridge

   默认桥接模式 在宿主机上创建一个虚拟的网桥 容器的网络接口连接到该网桥 并分配一个虚拟的子网 容器通过 NAT 访问外部网络

   

2. host

    容器使用宿主机的网络栈 不创建独立的网络接口 容器的网络直接映射到宿主机（与宿主机共享IP地址和网络端口） 

   

3. none

   容器没有网络功能 不分配网络接口 仅允许容器运行

   

4. container

   容器与另一个指定容器共享网络栈 共享网络接口 IP地址及端口 但文件系统 进程等保持隔离

   

5. macvlan

   容器分配一个独立的MAC地址 直接暴露在宿主机所在的物理网络中 容器与宿主机及其他设备通过物理网络通信

   

6. overlay

   用于 Docker 集群 （Swarm / Kubernetes）中 将多个主机的容器连接到一个虚拟网络 通过VXLAN（虚拟扩展LAN）在主机间建立跨主机网络

## Resource Quota

限制容器的计算/存储资源

### 限制CPU资源

在 docker 中 可通过 `--cpu-shares` /  `-c` 参数为容器设置CPU加权值 值越大 在满负载情况下 该容器获得的CPU时间越多 默认为1024

在 docker 中 可通过 ` --cpuset-cpus` 参数限制容器运行在哪个CPU节点上

### 限制内存资源

在 docker 中 可通过 ` --memory` /  `-m` 限制内存使用

## Auto Start

```bash
docker run -d --restart=always nginx
```



## Docker Harbor

Docker 的运行离不开可靠的镜像管理 虽然Docker官方提供公共镜像仓库 但从多方面考虑 构建私有环境中的Registry也是非常必要的 Harbor 是由 VMware 公司开源的企业级的 Docker Registry 管理项目 包括权限管理（RBAC）LDAP 日志审核 管理界面 自我注册 镜像复制 中文支持等

**官方网址** `https://github.com/goharbor/harbor`

### 自签发证书

```bash
# 在新的机器中进行操作 必须确保harbor与主机网络连接
mkdir /data/ssl -p
cd /data/ssl
openssl genrsa -out ca.key 3072
openssl req -new -x509 -days 3650 -key ca.key -out ca.pem
openssl genrsa -out harbor.key 3072
openssl req -new -key harbor.key -out harbor.csr
openssl x509 -req -in harbor.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out harbor.pem -days 3650

[ K8s - Harbor ~ ]: mkdir /data/ssl -p
[ K8s - Harbor ~ ]: cd /data/ssl
[ K8s - Harbor ssl ]: ls
[ K8s - Harbor ssl ]: openssl genrsa -out ca.key 3072
[ K8s - Harbor ssl ]: ls
ca.key
[ K8s - Harbor ssl ]: 
[ K8s - Harbor ssl ]: openssl req -new -x509 -days 3650 -key ca.key -out ca.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:BJ
Organization Name (eg, company) [Default Company Ltd]:archori
Organizational Unit Name (eg, section) []:k8s
Common Name (eg, your name or your server's hostname) []:harbor.cn
Email Address []:baizezhu@163.com
[ K8s - Harbor ssl ]: 
[ K8s - Harbor ssl ]: ls
ca.key  ca.pem
[ K8s - Harbor ssl ]: openssl genrsa -out harbor.key 3072
[ K8s - Harbor ssl ]: openssl req -new -key harbor.key -out harbor.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:BJ
Locality Name (eg, city) [Default City]:BJ
Organization Name (eg, company) [Default Company Ltd]:archori
Organizational Unit Name (eg, section) []:k8s
Common Name (eg, your name or your server's hostname) []:harbor.cn
Email Address []:baizezhu@163.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
[ K8s - Harbor ssl ]: ls
ca.key  ca.pem  harbor.csr  harbor.key
[ K8s - Harbor ssl ]: openssl x509 -req -in harbor.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out harbor.pem -days 3650
Certificate request self-signature ok
subject=C = CN, ST = BJ, L = BJ, O = archori, OU = k8s, CN = harbor.cn, emailAddress = baizezhu@163.com
[ K8s - Harbor ssl ]: ls
ca.key  ca.pem  ca.srl  harbor.csr  harbor.key  harbor.pem

# 不需要https可以忽略上述
```

### 安装 harbor

```bash
# 重启harbor
cd /data/install/harbor
docker-compose up -d
```



### 链接harbor

```bash
docker login harbor
# 链接要保证harbor与自签名证书的域名一样
```

### 镜像操作

```bash
# 打标签
docker tag image:version harbor.cn/library/image:version
# 上传镜像
docker push harbor.cn/library/image:version
# 获取镜像
docker pull harbor.cn/library/image:version
```



# Kubernetes



## VARIABLE DES.

此文档变量名含义

```bash
<PodName>                   # Pod 名称
<YAML_file>                 # Yaml 文件名称
<JSON_file>                 # Json 文件名称
<File_Name>                 # 文件名称
<ContainerName>             # Container 名称
<NameSpace_Name>            # NameSpace 名称
<ResourceQuota_Name>        # ResourceQuota Yaml 名称
<Node_name>                 # 节点名称
<Label_key>                 # 标签左值
<Label_Value>               # 标签右值
<taint_key>                 # 污点键
<taint_value>               # 污点值
<taint_effect>              # 污点排斥等级
<COMMAND>                   # 命令列表
<Deploy_Vision>             # 滚动更新版本号
<Service_Name>              # Service 资源名称
<Service_NameSpace>         # Service 资源名称空间
<Pod_UID>                   # Pod 资源的 UID
<Volume_Name>               # Volume 资源名称
<secret_Name>               # secret 资源名称
<cm_Name>                   # Configmap 资源名称
<Key>                       # 键
<Value>                     # 值
<Dir_path>                  # 目录路径
<secret_PASSWD>             # secret 资源密码
<String>                    # 字符串
```



**在安装Kubernetes之前**

```bash
# Kubernetes 相关的服务端口

Kube-apiserver 6443
etcd 2349 , 2380
kube-controller-manager 10257
kube-scheduler 10259
kubelet 10250
Kube-proxy 1024-65535
calico tcp 179 , 5743 , udp 4789 , 443
```

**配置阿里云docker和containerd在线源**

**配置免密登陆**

```bash
[ K8s - Lab 1 ~ ]: ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:8f5e2vg7Dw4JBCRAdUvD2lAObtn55Nh7lkiMn78JqPA root@localhost.localdomain
The key's randomart image is:
+---[RSA 4096]----+
|   .oo++B        |
|     ..O.=       |
|      ++* o      |
|     .. .%       |
|        S O      |
|         = = o   |
|    .   . B * o  |
|     o .   * O.. |
|      E    .Oo=+.|
+----[SHA256]-----+
[ K8s - Lab 1 ~ ]: ssh-copy-id root@node1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@node1's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@node1'"
and check to make sure that only the key(s) you wanted were added.
```

**安装Containerd 容器运行时**

```bash
# Kubernetes 中的容器运行时负责管理容器的生命周期和资源隔离，确保容器能够在节点上稳定地运行。其中，containerd 就是一个常用的容器运行时，它实现了 Kubernetes 规定的 CRI 接口，因此可以被 Kubernetes 作为容器运行时来使用。简单来说，就是 Kubernetes 需要容器运行时来管理和运行容器，而 containerd 就是 Kubernetes 中常用的一种容器运行时，能够让 Kubernetes 更加高效、稳定地运行容器。

yum install -y containerd.io-1.6.22*
```

**配置安装Kubernetes必要的源**

```bash
vim /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.31/rpm/repodata/repomd.xml.key
```

**Kubernetes主要包**

```bash
yum install -y kubelet-1.31.3 kubeadm-1.31.3 kubectl-1.31.3
systemctl enable kubelet --now
```

**在控制节点 使用kubeadm安装k8s**

```yaml
# 在控制节点 使用kubeadm生成配置文件
kubeadm config print init-defaults > kubeadm.yaml
[ K8s - Control ~ ]: find / -name "containerd.sock"
/run/containerd/containerd.sock

在生成的配置文件中 需要修改的参数如下：
advertiseAddress: 控制节点IP
criSocket: unix:///containerd.sock文件所在路径
name: 控制节点主机名
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kubernetesVersion: 1.31.3 安装kub版本号
在networking列表下添加
    podSubnet: 10.244.0.0/16
    
在最后追加一些东西
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd

# 查看安装时 需要哪些镜像
kubeadm config images list --kubernetes-version=1.31.3 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers

docker pull mage-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.31.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.31.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.31.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.31.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.11.3
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.10
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.15-0

# 除此之外
在 /etc/containerd/config.toml 中
找到 sandbox_image 字段 将后面的镜像也拉下来
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.7

# 最后 将拉下来的全部镜像同控制节点也要做一遍

# 在控制节点 使用kubeadm安装k8s
kubeadm init --config=kubeadm.yaml --ignore-preflight-errors=SystemVerification

# 除此之外
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 检查
[ K8s - Control ~ ]: kubectl get nodes
NAME      STATUS     ROLES           AGE   VERSION
control   NotReady   control-plane   27m   v1.31.3

[ K8s - Control ~ ]: kubectl get pods -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
coredns-fcd6c9c4-l9d57            0/1     Pending   0          27m
coredns-fcd6c9c4-x9v8t            0/1     Pending   0          27m
etcd-control                      1/1     Running   1          28m
kube-apiserver-control            1/1     Running   1          28m
kube-controller-manager-control   1/1     Running   1          28m
kube-proxy-pw484                  1/1     Running   0          27m
kube-scheduler-control            1/1     Running   1          28m
```

**在工作节点 安装k8s**

```bash
# 每次扩容节点 都需要在控制节点执行 生成内容以在工作节点上执行
kubeadm token create --print-join-command
# 添加参数 --ignore-preflight-errors=SystemVerification

# 查看节点状态
[ K8s - Control ~ ]: kubectl get nodes
NAME      STATUS     ROLES           AGE     VERSION
control   NotReady   control-plane   41m     v1.31.3
node1     NotReady   <none>          5m30s   v1.31.3
node2     NotReady   <none>          39s     v1.31.3

# 给节点更改标签
kubectl label nodes node1 node-role.kubernetes.io/work=work
[ K8s - Control ~ ]: kubectl get nodes
NAME      STATUS     ROLES           AGE     VERSION
control   NotReady   control-plane   44m     v1.31.3
node1     NotReady   work            8m40s   v1.31.3
node2     NotReady   work            3m49s   v1.31.3


```

**安装Kubernetes网络插件**

```bash
# 在所有节点上
[ K8s - Node 1 ~ ]: ctr -n=k8s.io images import calico.tar.gz 

# 在控制节点上 需要改的 calico.yaml字段
- name: IP_AUTODETECTION_METHOD
              value: "interface=ens160"
              
# 然后执行
kubectl apply -f calico.yaml

# 在node1 / node2
ctr -n k8s.io images import busybox-1-28.tar.gz

# 在控制节点验证
kubectl run busybox --image docker.io/library/busybox:1.28 --image-pull-policy=IfNotPresent --restart=Never --rm -it busybox -- sh
If you don't see a command prompt, try pressing enter.
/ # ping www.baidu.com
PING www.baidu.com (180.101.50.188): 56 data bytes
64 bytes from 180.101.50.188: seq=0 ttl=127 time=24.235 ms
64 bytes from 180.101.50.188: seq=1 ttl=127 time=24.392 ms
^C
--- www.baidu.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 24.235/24.313/24.392 ms
/ # 

# 此结果说明网络插件没有问题

```

**延长K8s证书时间**

```bash
# 在控制节点上执行

openssl x509 -in /etc/kubernetes/pki/ca.crt -noout -text  |grep Not
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text  |grep Not

此略

```

## Pod

Pod 是 Kubernetes 中管理的最小单元。pod可以通过yaml文件创建，也可以使用kubectl run创建。

### Pod工作模式

#### 自主式Pod

所谓自主式pod ，就是直接定义一个Pod资源，如：

```bash
# 使用yaml文件创建pod
```

```yaml
apiVersion: v1  #api版本
kind: Pod       #创建的资源
metadata:    
  name: tomcat-test  #Pod的名字
  namespace: default   #Pod所在的名称空间
  labels:
    app:  tomcat     #Pod具有的标签
spec:
  containers:
  - name:  tomcat-java   #Pod里容器的名字
    ports:
    - containerPort: 8080  #容器暴露的端口
    image: xianchao/tomcat-8.5-jre8:v1  #容器使用的镜像
    imagePullPolicy: IfNotPresent    #镜像拉取策略
```

```bash

```

**导入镜像**

```bash
[ K8s - Control home ]: scp xianchao-nginx.tar.gz root@node1:/home
xianchao-nginx.tar.gz      100%  131MB  41.5MB/s   00:03    
[ K8s - Control home ]: scp xianchao-tomcat.tar.gz  root@node1:/home
xianchao-tomcat.tar.gz     100%  106MB  44.2MB/s   00:02    
[ K8s - Control home ]: scp xianchao-nginx.tar.gz root@node2:/home
xianchao-nginx.tar.gz      100%  131MB  43.8MB/s   00:02    
[ K8s - Control home ]: scp xianchao-tomcat.tar.gz  root@node2:/home
xianchao-tomcat.tar.gz     100%  106MB  39.7MB/s   00:02

ssh node1
[ K8s - Node 1 home ]: ctr -n=k8s.io images import xianchao-tomcat.tar.gz 
unpacking docker.io/xianchao/tomcat-8.5-jre8:v1 (sha256:14dae4798a335e2925e4ee07b3ccd519a67faab2a9155adeb76a4556478dd8d7)...done
# 解压镜像
ssh node2
[ K8s - Node 2 home ]: ctr -n=k8s.io images import xianchao-tomcat.tar.gz 
unpacking docker.io/xianchao/tomcat-8.5-jre8:v1 (sha256:14dae4798a335e2925e4ee07b3ccd519a67faab2a9155adeb76a4556478dd8d7)...done
```

```bash
kubectl apply -f pod-tomcat.yaml

kubectl get pods
[ K8s - Control pod ]: kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
tomcat-test   1/1     ContainerCreating   0          13m
```

#### 控制器管理的Pod

常用的控制器有 Replaicast , Deployment , Job , CronJob , Daemonest , StateFulest

控制器管理的Pod可以保证Pod始终在指定的副本数运行

如 Deployment：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  labels:
    app: nginx-deploy
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: my-nginx
        image: xianchao/nginx:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80

```



## YAML

运行 Kubernetes 帮助命令

`kubectl explain 资源信息`

`kubectl explain pod`

```bash
[ K8s - Control pod ]: kubectl explain pod
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.
    
FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status	<PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

### 使用 YAML 文件规范创建 Pod

```yaml
apiVersion: v1       <string>
kind: Pod            <string>
metadata:            <object>
  annotations:         <map[string]string> # 注解类型，包含多个键值对，无实际作用
  deletionGracePeriodSeconds:  <integer>
  labels:              <map[string]string> # 标签类型，包含多个键值对，常用于表示Pod运行的属性,可包含多个自定义键值对
  name:                <string>            # 命名
  namespace: default   <string>            # 名称空间
spec:                <PodSpec>
  activeDeadlineSeconds: <integer>         # Pod 的最长生存时间
  affinity:            <Affinity>          # Pod 亲和性
  dnsConfig:           <PodDNSConfig>      # Pod DNS
  containers:          <[]Container>       # 必须的字段，重要的，这个是对象列表字段，在子列表中需要添加 - 指示列表
  - name:                <string>          # 必须的字段，指定容器名称
    image:               <string>          # 指定容器所用镜像
    imagePullPolicy:     <string>          # 一般必须字段，指定镜像拉取策略。参数 # Always 不检查本地是否有镜像，总是从镜像仓库中拉取镜像 # IfNotPresent 如果本地存在指定镜像则直接使用 # Never 不检查本地是否有镜像，总是使用本地镜像
    ports:               <[]ContainerPort> # 对象列表字段，指定容器端口属性
    - containerPort        <integer>       # 必须的字段，指定容器端口号
  
```

> [!NOTE]
>
> 对于 labels 标签字段，该字段自定义键值对，目的标记Pod的标签属性，方便在后续查找中分类。
>
> ```bash
> [ K8s - Control pod ]: kubectl get pods -l Occu=Archori
> NAME     READY   STATUS    RESTARTS   AGE
> tomcat   1/1     Running   0          20m
> 
> # 查找标签为 Occu 值为 Archori 的 Pod
> ```
>
> 对于 namespace 命名空间字段，也可以添加 -n 参数在后续查找中分类
>
> ```bash
> [ K8s - Control pod ]: kubectl get pods -n default
> NAME     READY   STATUS    RESTARTS   AGE
> tomcat   1/1     Running   0          20m
> ```
>
> 

**示例 基于Pod管理器的 YAML 文件**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tomcat
  labels:
    app: tomcat
    Occu: Archori
    namespace: default
spec:
  containers:
  - name: tomcat-test
    image: docker.io/xianchao/tomcat-8.5-jre8:v1
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8080
```

```bash
[ K8s - Control pod ]: kubectl apply -f test.yaml 
pod/tomcat created
[ K8s - Control pod ]: kubectl get pods
tomcat                      1/1     Running            0          14s
```



## Control

### 创建 Pod

#### 通过命令行创建

```bash
kubectl run <PodName> <参数>
```

```bash
kubectl run tomcat --image=xianchao/tomcat-8.5-jre8:v1 --image-pull-policy='IfNotPresent' --port=8080
pod/tomcat created

[ K8s - Control pod ]: kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
tomcat   1/1     Running   0          6s
```



### 进入 Pod

**为创建的Pod分配伪终端**

```bash

kubectl exec -it <PodName> -- /bin/bash

# 这是针对 Pod 中只含有一个 Container 而言，命令执行后直接进入缺省的容器，对于 Pod 中含有多个 Containers 则添加 -c 参数指定进入的容器，如下。

kubectl exec -it <PodName> -c <ContainerName> -- /bin/bash
```

```bash
[ K8s - Control pod ]: kubectl get pods
tomcat                      1/1     Running            0          14s
[ K8s - Control pod ]: kubectl exec -it tomcat -- /bin/bash
bash-4.4#

# 规范写法

kubectl exec -it tomcat -c tomcat-test -- /bin/bash

[ K8s - Control pod ]: kubectl exec -it tomcat -c tomcat-test -- /bin/bash
bash-4.4#
```

### 删除 Pod

#### 直接删除 Pod

```bash
kubectl delete pods <PodName>
```

```bash
[ K8s - Control pod ]: kubectl delete pods tomcat
pod "tomcat" deleted
```

#### 通过YAML文件删除 Pod

```bash
kubectl delete -f <YAML_file>
```

```bash
[ K8s - Control pod ]: kubectl delete -f test.yaml 
pod "tomcat" deleted
```



### 信息管理

#### 查看 Pod 日志

```bash
kubectl logs <PodName>
# 该命令显示指定 Pod 中所有 Containers 的日志
```

```bash
[ K8s - Control pod ]: kubectl logs tomcat
27-Apr-2025 02:43:25.578 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version:        Apache Tomcat/8.5.34
27-Apr-2025 02:43:25.587 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Sep 4 2018 22:28:22 UTC
27-Apr-2025 02:43:25.587 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server number:         8.5.34.0
27-Apr-2025 02:43:25.587 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
27-Apr-2025 02:43:25.588 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log 
OS Version:            5.14.0-284.11.1.el9_2.x86_64
...
...
... 此处省略更多 ...
```

#### 查看 Pod 描述

```bash
kubectl describe pods <PodName>
```

```bash
[ K8s - Control pod ]: kubectl describe pods tomcat
Name:             tomcat
Namespace:        default
Priority:         0
Service Account:  default
Node:             node1/192.168.152.101
Start Time:       Sun, 27 Apr 2025 10:43:22 +0800
Labels:           Occu=Archori
                  app=tomcat
                  namespace=default
...
...
... 此处省略更多 ...

# 此描述可以看到指定 Pod 的所有信息和指定 Pod 中运行的 Containers 的所有信息
```

#### 查看 Pod 调度

```bash
kubectl get pods -owide
```

```bash
[ K8s - Control pod ]: kubectl get pods -owide
NAME                        READY   STATUS             RESTARTS   AGE     IP               NODE    NOMINATED NODE   READINESS GATES
nginx-test-5fbcf9f6-g8s4j   0/1     ImagePullBackOff   0          4d21h   10.244.104.4     node2   <none>           <none>
nginx-test-5fbcf9f6-t4d9j   0/1     ImagePullBackOff   0          4d21h   10.244.166.133   node1   <none>           <none>
tomcat                      1/1     Running            0          36m     10.244.166.134   node1   <none>           <none>
```

#### 查看 Pod 标签

```bash
kubectl get pods --show-labels
```

```bash
[ K8s - Control pod ]: kubectl get pods --show-labels
NAME                        READY   STATUS              RESTARTS   AGE     LABELS
tomcat                      0/1     ContainerCreating   0          10s     Occu=Archori,app=tomcat,namespace=default
```

#### 查看所有名称空间下的 Pod

```bash
kubectl get pods --all-namespaces
```

```bash
[ K8s - Control pod-1 ]: kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS             RESTARTS      AGE
default       tomcat                                     1/1     Running            0             6d20h
kube-system   calico-kube-controllers-695bcfd99c-jtpnv   0/1     ImagePullBackOff   0             37d
kube-system   calico-node-8pr7r                          1/1     Running            1 (61d ago)   77d
kube-system   calico-node-9s475                          1/1     Running            3 (19d ago)   77d
kube-system   calico-node-v5lzr                          1/1     Running            1 (61d ago)   77d
kube-system   coredns-fcd6c9c4-9fn8c                     1/1     Running            0             41d
kube-system   coredns-fcd6c9c4-cvpxp                     1/1     Running            0             37d
kube-system   etcd-control                               1/1     Running            7 (19d ago)   105d
kube-system   kube-apiserver-control                     1/1     Running            7 (19d ago)   105d
kube-system   kube-controller-manager-control            1/1     Running            9 (19d ago)   105d
kube-system   kube-proxy-pw484                           1/1     Running            6 (19d ago)   105d
kube-system   kube-proxy-wfrwk                           1/1     Running            2 (61d ago)   105d
kube-system   kube-proxy-xv6z6                           1/1     Running            2 (61d ago)   105d
kube-system   kube-scheduler-control                     1/1     Running            9 (19d ago)   105d
test          pod-test                                   1/1     Running            0             8d
```

#### 查看 Pod 细节信息

```bash
kubectl describe pods <PodName>
```

```bash
[ K8s - Control pod-nodeName ]: kubectl describe pods demo-pod
Name:             demo-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             node1/192.168.152.101
Start Time:       Tue, 06 May 2025 16:46:45 +0800
Labels:           Occu=Archori
                  app=myapp
                  env=dev
Annotations:      cni.projectcalico.org/containerID: 4a0987870088cc8542a78b0a85cdeae15311554c23fa21ff3b95aa44f6eec3de
                  cni.projectcalico.org/podIP: 10.244.166.138/32
                  cni.projectcalico.org/podIPs: 10.244.166.138/32
Status:           Pending
IP:               10.244.166.138
IPs:
  IP:  10.244.166.138
Containers:
  tomcat-pod-java:
    Container ID:   containerd://4005b1558b99bdc6b7b40fe05a1716b3e71d06d240f11bcfea1f97d6b9ef3f94
    Image:          tomcat:8.5-jre8-alpine
    Image ID:       sha256:8b8b1eb786b54145731e1cd36e1de208d10defdbb0b707617c3e7ddb9d6d99c8
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 06 May 2025 16:48:30 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tsvm4 (ro)
  busybox:
    Container ID:  
    Image:         busybox:latest
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      sleep 3600
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tsvm4 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-tsvm4:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason                  Age                  From     Message
  ----     ------                  ----                 ----     -------
  Warning  FailedCreatePodSandBox  3m23s                kubelet  Failed to create pod sandbox: rpc error: code = Unknown desc = failed to setup network for sandbox "f7b46db93598273d57cb4bb6295da4ae31a696f5d89b92bbb35825b62fd12880": plugin type="calico" failed (add): error getting ClusterInformation: connection is unauthorized: Unauthorized
  Normal   SandboxChanged          99s (x9 over 3m22s)  kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal   Pulled                  98s                  kubelet  Container image "tomcat:8.5-jre8-alpine" already present on machine
  Normal   Created                 98s                  kubelet  Created container tomcat-pod-java
  Normal   Started                 98s                  kubelet  Started container tomcat-pod-java
  Warning  Failed                  56s                  kubelet  Failed to pull image "busybox:latest": failed to pull and unpack image "docker.io/library/busybox:latest": failed to resolve reference "docker.io/library/busybox:latest": pulling from host vh3bm52y.mirror.aliyuncs.com failed with status code [manifests latest]: 403 Forbidden
  Warning  Failed                  56s                  kubelet  Error: ErrImagePull
  Normal   BackOff                 55s                  kubelet  Back-off pulling image "busybox:latest"
  Warning  Failed                  55s                  kubelet  Error: ImagePullBackOff
  Normal   Pulling                 42s (x2 over 98s)    kubelet  Pulling image "busybox:latest"
```

#### 查看 Node 中存在镜像

```bash
crictl images ls
```

```bash
[ K8s - Node 1 ~ ]: crictl images ls
WARN[0000] image connect using default endpoints: [unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
IMAGE                                                            TAG                 IMAGE ID            SIZE
docker.io/calico/cni                                             v3.26.1             9dee260ef7f59       210MB
docker.io/calico/kube-controllers                                v3.26.1             1919f2787fa70       75.1MB
docker.io/calico/node                                            v3.26.1             8065b798a4d67       248MB
docker.io/calico/pod2daemon-flexvol                              v3.26.1             092a973bb20ee       14.9MB
docker.io/library/busybox                                        1.28                8c811b4aec35f       1.36MB
docker.io/library/busybox                                        latest              388056c9a6838       1.45MB
docker.io/library/tomcat                                         8.5-jre8-alpine     8b8b1eb786b54       110MB
docker.io/xianchao/tomcat-8.5-jre8                               v1                  4ac473a3dd922       111MB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns      v1.11.3             c69fa2e9cbf5f       18.6MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy   v1.31.3             9c4bd20bd3676       30.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause        3.7                 221177c6082a8       311kB
```





### 查看Pod状态

使用 `kubectl get pods`  STATUS 字段，表示当前 Pod 状态

#### Phase 状态

```bash
STATUS

Pending        # Pod被接受，但容器未创建完成。
Running        # Pod被接受，并成功创建。
Succeeded      # Pod中所有容器成功终止，例如一次性job。
Failed         # Pod中所有容器被终止，但至少一个容器失败。
Unknown        # 因为某种原因无法获取 Pod 状态。
```

#### 容器级别状态

```bash
STATUS

CrashLoopBackOff
# 容器启动崩溃，并不断重启，可能程序错误，配置问题导致。

ImagePullBackOff
# 拉取镜像失败，Kubernetes 会尝试重新拉取。

ErrImagePull
# 拉取镜像失败，但不会重试。通常是镜像地址问题。

CreateContainerConfigError
# 创建容器配置失败，通常是 YAML 配置错误。

RunContainerError
# 容器启动失败，镜像有问题或者启动命令错误。

OOMKilled
# 容器因超出内存限制而被系统杀死。

ContainerCreating
# 容器正在被创建中。

Terminating
# Pod 正在被删除，Kubernetes 在“优雅”地终止容器。

NodeLost
# 所在节点不可达，Kubernetes 无法确认状态。
```



### 重启策略

Pod 的重启策略 (RestartPolicy) 应用于 Pod 中的所有 Container 。当某个 Container 异常退出或健康检查失败时，kubelet 将根据重启策略字段进行相应操作。



描述

```bash
restartPolicy
```

路径

```yaml
spec:
  restartPolicy:
```

查询

```bash
kubectl explain pod.spec.restartPolicy
```

值

```yaml
spec:
  restartPolicy: # Always / Never / OnFailure
 
Always           # 只要容器异常，就执行重启策略（默认选项）
Never            # 无论什么状况，总不重启
OnFailure        # 当容器终止且退出码不为0时，执行重启
```







## NameSpace

Kubernetes 支持多个虚拟机群，它们底层依赖同一个物理集群，Kubernetes使用 **命名空间 (NameSpace) ** 来区分不同的虚拟机群。

命名空间是k8s集群级别的资源，可以给不同的用户、租户、环境或项目创建对应的命名空间，例如，可以为test、devlopment、production环境分别创建各自的命名空间。

### 控制

#### 创建命名空间

```bash
kubectl create namespace <NameSpace_Name>
kubectl create ns <NameSpace_Name>
```

```bash
[ K8s - Control ~ ]: kubectl create namespace test
namespace/test created
[ K8s - Control ~ ]: kubectl get namespace
NAME              STATUS   AGE
default           Active   97d
kube-node-lease   Active   97d
kube-public       Active   97d
kube-system       Active   97d
test              Active   7s
```

#### 使用YAML文件进行资源配额

使用 NameSpace 可以限制在名称空间下 Pod 使用的资源配额，方便集中/分类管理。

##### **查看帮助命令**

```bash
kubectl explain resourcequota
```

```bash
[ K8s - Control pod-1 ]: kubectl explain resourcequota
KIND:       ResourceQuota
VERSION:    v1

DESCRIPTION:
    ResourceQuota sets aggregate quota restrictions enforced per namespace
    
FIELDS:
  apiVersion	<string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind	<string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata	<ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec	<ResourceQuotaSpec>
    Spec defines the desired quota.
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status	<ResourceQuotaStatus>
    Status defines the actual enforced quota and its current usage.
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
```

##### **编写YAML文件**

```yaml
apiVersion: v1        # apiversion
kind: ResourceQuota   # 资源类型为 ResourceQuota
metadata:             # 必要字段
  name: mem-cpu-quota # 为当前资源配额命名
  namespace: test     # 作用的命名空间
spec:                 # 必要字段
  hard:               # 对硬件操作
    requests.cpu: 2
    requests.memory: 2Gi
    limits.cpu: 4
    limits.memory: 4Gi
```

> [!NOTE]
>
> requests 和 limits 有什么不同？
>
> requests **调度时**使用（决定 Pod 能否调度到某个节点）
>
> limits **运行时**使用（决定 Pod 实际能用多少资源）

##### 执行资源配额

```bash
kubectl apply -f <ResourceQuota_Name>
```

```bash
[ K8s - Control pod-1 ]: kubectl apply -f namespace-quota.yaml 
resourcequota/mem-cpu-quota created
```

##### 查看资源配额

```bash
kubectl get resourcequota -n <NameSpace_Name>
```

```bash
[ K8s - Control pod-1 ]: kubectl get resourcequota -n test
NAME            AGE    REQUEST                                     LIMIT
mem-cpu-quota   106s   requests.cpu: 0/2, requests.memory: 0/2Gi   limits.cpu: 0/4, limits.memory: 0/4Gi
```

**示例**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-test             # 指定 Pod 名称
  namespace: test            # 指定命名空间
  labels:
    Occu: Archori
    app: tomcat-pod-test
spec:
  containers:
  - name: tomcat-test        # 指定容器名称
    image: docker.io/xianchao/tomcat-8.5-jre8:v1
    imagePullPolicy: IfNotPresent
    ports:                   # 端口策略
    - containerPort: 8080    # 指定端口号
    resources:               # 资源配额
      limits:                # 运行限制
        memory: "2Gi"
        cpu: "2"             # 无单位为大核
      requests:              # 部署限制
        memory: "100Mi"
        cpu: "500m"          # 1000m为1核
```

```bash
[ K8s - Control pod-1 ]: kubectl apply -f pod-test.yaml 
pod/pod-test created
[ K8s - Control pod-1 ]: kubectl get pods -n test
NAME       READY   STATUS    RESTARTS   AGE
pod-test   1/1     Running   0          8s
[ K8s - Control pod-1 ]: kubectl get resourcequota -n test
NAME            AGE   REQUEST                                            LIMIT
mem-cpu-quota   41m   requests.cpu: 500m/2, requests.memory: 100Mi/2Gi   limits.cpu: 2/4, limits.memory: 2Gi/4Gi
```

> [!NOTE]
>
> 关于资源限制层级的提示：
>
> 对于使用配置文件创建的限制，作用在 Pod 整体上，限制 Pod 内所有 Containers 和的资源。
>
> 对于使用配置文件中 spec-containers-resources 字段的限制，作用在单个 Container 上，限制单个 Container 的部署和运行。
>
> 对于作用在 Containers 中的资源限制不可超过命名空间资源限额文件作用在 Pod 上的限制。
>
> 如果所属的 NameSpace 中对资源有限制，那么使用配置文件创建 Pod 中的 Containers 中必须对资源进行限制，那么 resource 这个字段就是必须的。



## Label

标签字段提供了一个 **自定义键值对列表** 。它被关联在对象上，用于表示对象的不同属性，如服务类型，服务版本等。标签可以在初始化的时候定义，也可以在运行期间重构。

在 Kubernetes 中，大部分类型都可以使用标签字段。

### 为Pod添加标签

```bash
kubectl label pods <PodName> <key>=<value>
```



## Node scheduler

当 Pod 被创建的时候，scheduler 会将 Pod 调度到合适的任意节点。当我们需要调度到指定节点时，可以使用 **nodeName** 或 **nodeSelector** 实现。

```bash
# Prepare Work
[ K8s - Control home ]: scp ./tomcat.tar.gz root@node1:/home
tomcat.tar.gz               100%  105MB  46.7MB/s   00:02    
[ K8s - Control home ]: scp ./tomcat.tar.gz root@node2:/home
tomcat.tar.gz               100%  105MB  42.2MB/s   00:02    
[ K8s - Control home ]: ssh node1
[ K8s - Node 1 ~ ]: ctr -n=k8s.io images import /home/tomcat.tar.gz 
unpacking docker.io/library/tomcat:8.5-jre8-alpine (sha256:463a0b1de051bff2208f81a86bdf4e7004eb68c0edfcc658f2e2f367aab5e342)...done
[ K8s - Control home ]: ssh node2 ctr -n=k8s.io images import /home/tomcat.tar.gz
unpacking docker.io/library/tomcat:8.5-jre8-alpine (sha256:463a0b1de051bff2208f81a86bdf4e7004eb68c0edfcc658f2e2f367aab5e342)...done
[ K8s - Control item ]: scp ./busybox.tar.gz root@node1:/home
busybox.tar.gz              100% 1425KB  35.2MB/s   00:00    
[ K8s - Control item ]: ssh node1 ctr -n=k8s.io images import /home/busybox.tar.gz
unpacking docker.io/library/busybox:latest (sha256:2d86744fc4e303fbf4e71c67b89ee77cc6c60e9315cbd2c27f50e85b2d866450)...done
[ K8s - Control item ]: scp ./busybox.tar.gz root@node2:/home
busybox.tar.gz              100% 1425KB  39.2MB/s   00:00    
[ K8s - Control item ]: ssh node2 ctr -n=k8s.io images import /home/busybox.tar.gz
unpacking docker.io/library/busybox:latest (sha256:2d86744fc4e303fbf4e71c67b89ee77cc6c60e9315cbd2c27f50e85b2d866450)...done
```



### nodeName

使用 **nodeName** 字段指定调度节点

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  namespace: default
  labels:
    app: myapp
    env: dev
    Occu: Archori
spec:
  nodeName: node1
  containers:
    - name: tomcat-pod-java
      ports:
      - containerPort: 8080
      image: tomcat:8.5-jre8-alpine
      imagePullPolicy: IfNotPresent
    - name: busybox
      image: busybox:latest
      command:
        - "/bin/sh"
        - "-c"
        - "sleep 3600"
```

```bash
[ K8s - Control pod-nodeName ]: kubectl apply -f pod-node.yaml 
pod/demo-pod created
[ K8s - Control pod-nodeName ]: kubectl get pods
NAME       READY   STATUS              RESTARTS   AGE
demo-pod   0/2     ContainerCreating   0          8s
tomcat     1/1     Running             0          6d21h
```

### nodeSelector

使用 **nodeSelector** 字段进行节点选择调度可以通过 labels 标签属性选择 node 节点。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod-1
  namespace: default
  labels:
    app: myapp
    env: dev
    Occu: Archori
spec: 
  nodeSelector:        # 节点选择调度字段
    disk: ceph         # 标签
  containers:
  - name: tomcat-pod-java
    ports:
    - containerPort: 8080
    image: tomcat:8.5-jre8-alpine
    imagePullPolicy: IfNotPresent
```

#### 查看节点标签

```bash
kubectl get nodes --show-labels
```

```bash
[ K8s - Control pod-nodeName ]: kubectl get nodes --show-labels
NAME      STATUS   ROLES           AGE    VERSION   LABELS
control   Ready    control-plane   105d   v1.31.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=control,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=
node1     Ready    work            105d   v1.31.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node1,kubernetes.io/os=linux,node-role.kubernetes.io/work=work
node2     Ready    work            105d   v1.31.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node2,kubernetes.io/os=linux,node-role.kubernetes.io/work=work
```

#### 添加节点标签

```bash
kubectl label nodes <Node_name> <Label_key>=<Label_Value>
```

```bash
[ K8s - Control pod-nodeName ]: kubectl label nodes node1 disk=ceph
node/node1 labeled
```

#### 删除节点标签

```bash
kubectl label nodes <Node_name> <label_key>-
```

```bash
[ K8s - Control pod-nodeName ]: kubectl label nodes node1 disk-
node/node1 unlabeled
```

## Affinity

Affinity 亲和性调度策略提供了复杂,繁琐但极其强大的调度解决方案。



描述

```bash
affinity
```

路径

```yaml
spec:
  affinity:
```

查询

```bash
kubectl explain pod.spec.affinity
```



### Node 亲和性



描述

```bash
nodeAffinity
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity
```



#### 偏好亲和性



描述

```bash
preferredDuringSchedulingIgnoredDuringExecution
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution
```



##### 节点偏好



描述

```bash
preference
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution.preference
```



###### 标签匹配



描述

```bash
matchExpressions
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
          matchExpressions:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution.preference.matchExpressions
```

值

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
          matchExpressions:
          - key:         # 标签键
            operator:    # 操作符
            values:      # 标签值
            - <value1>
            - <value2>
            - <value...>
            
## 其中操作符有

DoesNotExist   # 键不存在
Exists         # 键存在
Gt             # 键存在，且值大于指定值
In             # 键存在，且值存在于列表
Lt             # 键存在，且值小于指定值
NotIn          # 键存在，且值不在于列表
```







##### 权重

描述

```bash
weight
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
        weight:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.preferredDuringSchedulingIgnoredDuringExecution.weight
```

值

```yaml
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
          matchExpressions:
          - key:         # 标签键
            operator:    # 操作符
            values:      # 值列表
            - <value1>
            - <value2>
            - <value...>
        weight: <0~100>  # 权重值 值越大权重越高
```





#### 强制亲和性



描述

```bash
requiredDuringSchedulingIgnoredDuringExecution
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      requireDuringSchedulingIgnoreDuringExecution:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution
```



##### 节点选择

描述

```bash
nodeSelectorTerms
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms
```



###### 标签匹配



描述

```bash
matchExpressions
```

路径

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
```

查询

```bash
kubectl explain pod.spec.affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms.matchExpressions
```

值

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key:          # 标签键
            operator:     # 操作符
            values:       # 值列表
            - <value1>
            - <value2>
            - <value...>
        
## 其中操作符有

DoesNotExist   # 键不存在
Exists         # 键存在
Gt             # 键存在，且值大于指定值
In             # 键存在，且值存在于列表
Lt             # 键存在，且值小于指定值
NotIn          # 键存在，且值不在于列表
```





### Pod 亲和性



描述

```bash
podAffinity
```

路径

```yaml
spec:
  affinity:
    podAffinity:
```

查询

```bash
kubectl explain pod.spec.podAffinity
```





#### 偏好亲和性









#### 强制亲和性



描述

```bash
requiredDuringSchedulingIgnoredDuringExecution
```

路径

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
```

查询

```bash
kubectl explain pod.spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution
```



##### 标签选择器



描述

```bash
labelSelector
```

路径

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
```

查询

```bash
kubectl explain pod.spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution.labelSelector
```



###### 标签匹配



描述

```bash
matchExpressions
```

路径

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
        matchExpressions:
```

查询

```bash
kubectl explain pod.spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution.labelSelector.matchExpressions
```

值

```yaml
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
        matchExpressions:
        - key:              # 标签键
          operator:         # 匹配符
          values:           # 值列表
          - <value1>
          - <value2>
          - <value..>
          
          
## 匹配符包含

DoesNotExist   # 键不存在
Exists         # 键存在
In             # 键存在，且值存在于列表
NotIn          # 键存在，且值不在于列表
```





##### 拓扑范围选择器



描述

```bash
topologyKey
```

路径

```yaml
spec:
  affinity:
    PodAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        topologyKey:
```

查询

```bash
kubectl explain pod.spec.affinity.podAffinity.requiredDuringSchedulingIgnoredDuringExecution.topologyKey
```

功能

```bash
此字段目的使用 Kubernetes 默认的标签区分 节点，机架，区域

kubernetes.io/hostname         # 同一个节点
topology.kubernetes.io/zone    # 同一个可用区
topology.kubernetes.io/region  # 同一个地域

更多的，还可以使用自定义的 Node Label 如 rack-id 等
重要的，匹配机制在于，标签的键值相同认为是同一个节点，区，域。
此字段的功能在于用到 亲和性/反亲和性 中，匹配节点所在节点/区/域作亲和/反亲和。

例如：
matchExpressions 字段 匹配到
node1
node2
而且
node1
node2
有相同标签 zone=foo1
node3
node4
有相同标签 zone=foo2
使用字段 topologyKey:zone
亲和性为 podAntiAffinity
那么节点调度时就会在
node3
node4
中选择
```





### Pod 反亲和性



描述

```bash
podAntiAffinity
```

路径

```yaml
spec:
  affinity:
    podAntiAffinity:
```

查询

```bash
kubectl explain pod.spec.affinity.podAntiAffinity
```





#### 偏好反亲和



描述

```bash
preferredDuringSchedulingIgnoredDuringExecution
```

路径

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
```

查询

```bash
kubectl explain pod.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution
```



###### 规则



描述

```bash
podAffinityTerm
```

路径

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - podAffinityTerm:
```

查询

```bash
kubectl explain pod.spec.affinity.podAntiAffinity.preferredDuringSchedulingIgnoredDuringExecution.podAffinityTerm
```

值

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - podAffinityTerm:       # 匹配规则
          labelSelector:       # 标签匹配
            matchExpressions:  # 表达式匹配
            - key:             # 标签键
              operator:        # 操作符
              values:          # 标签值
              - <value1>
              - <value2>
              - <values>
          topologyKey: zone    # 拓扑范围
        weight:                # 权重
```





#### 强制反亲和



描述

```bash
requiredDuringSchedulingIgnoredDuringExecution
```

路径

```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
```

查询

```bash
kubectl explain pod.spec.affinity.podAntiAffinity.requiredDuringSchedulingIgnoredDuringExecution
```



## Taint & Toleration

Kubernetes 中的**污点（Taint）**和**容忍度（Toleration）**是一套用于控制 **Pod 是否能调度到某个 Node** 的机制。



### 污点

污点是给予 node 节点的描述，污点的作用在于限制 Pod 调度到有污点的 node 上。

#### 添加污点

```bash
kubectl taint nodes <Node_name> <taint_key>=<taint_value>:<taint_effect>
```

```bash
kubectl taint nodes node1 Env=dev:NoSchedule
node/node1 tainted

[ K8s - Control ~ ]: kubectl describe nodes node1
Name:               node1
Roles:              work
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=node1
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/work=work
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.152.101/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 10.244.166.128
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 20 Jan 2025 20:52:19 +0800
Taints:             Env=dev:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  node1
  AcquireTime:     <unset>
  RenewTime:       Mon, 12 May 2025 16:44:10 +0800
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Wed, 05 Mar 2025 18:33:11 +0800   Wed, 05 Mar 2025 18:33:11 +0800   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Mon, 12 May 2025 16:41:57 +0800   Sun, 30 Mar 2025 15:01:48 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Mon, 12 May 2025 16:41:57 +0800   Sun, 30 Mar 2025 15:01:48 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Mon, 12 May 2025 16:41:57 +0800   Sun, 30 Mar 2025 15:01:48 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Mon, 12 May 2025 16:41:57 +0800   Sun, 30 Mar 2025 15:01:48 +0800   KubeletReady                 kubelet is posting ready status
Addresses:
  InternalIP:  192.168.152.101
  Hostname:    node1
Capacity:
  cpu:                4
  ephemeral-storage:  17394Mi
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             15713Mi
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  16415037823
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             15613Mi
  pods:               110
System Info:
  Machine ID:                 42294656dcee4f89bf8023caa069f1e3
  System UUID:                99aa4d56-e27b-1d96-7ab1-aa2ebe09af5d
  Boot ID:                    482b2858-c661-4fb2-8b20-a25178e63df9
  Kernel Version:             5.14.0-284.11.1.el9_2.x86_64
  OS Image:                   Red Hat Enterprise Linux 9.2 (Plow)
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.6.22
  Kubelet Version:            v1.31.3
  Kube-Proxy Version:         v1.31.3
PodCIDR:                      10.244.1.0/24
PodCIDRs:                     10.244.1.0/24
Non-terminated Pods:          (2 in total)
  Namespace                   Name                 CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                 ------------  ----------  ---------------  -------------  ---
  kube-system                 calico-node-8pr7r    250m (6%)     0 (0%)      0 (0%)           0 (0%)         83d
  kube-system                 kube-proxy-wfrwk     0 (0%)        0 (0%)      0 (0%)           0 (0%)         111d
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests   Limits
  --------           --------   ------
  cpu                250m (6%)  0 (0%)
  memory             0 (0%)     0 (0%)
  ephemeral-storage  0 (0%)     0 (0%)
  hugepages-1Gi      0 (0%)     0 (0%)
  hugepages-2Mi      0 (0%)     0 (0%)
Events:              <none>
```



#### 删除污点

```bash
kubectl taint nodes <Node_name> node-type:<taint_effect>
```

```bash
[ K8s - Control pod-probe-status ]: kubectl taint nodes node1 Env-
node/node1 untainted
[ K8s - Control pod-probe-status ]: kubectl describe node node1 | grep Taints
Taints:             <none>
```





#### 查看污点

```bash
[ K8s - Control pod-probe-status ]: kubectl get nodes -o wide
NAME      STATUS   ROLES           AGE    VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                              KERNEL-VERSION                 CONTAINER-RUNTIME
control   Ready    control-plane   188d   v1.31.3   192.168.152.100   <none>        Red Hat Enterprise Linux 9.2 (Plow)   5.14.0-284.11.1.el9_2.x86_64   containerd://1.6.22
node1     Ready    work            188d   v1.31.3   192.168.152.101   <none>        Red Hat Enterprise Linux 9.2 (Plow)   5.14.0-284.11.1.el9_2.x86_64   containerd://1.6.22
node2     Ready    work            188d   v1.31.3   192.168.152.102   <none>        Red Hat Enterprise Linux 9.2 (Plow)   5.14.0-284.11.1.el9_2.x86_64   containerd://1.6.22
```

```bash
[ K8s - Control pod-probe-status ]: kubectl describe node control | grep -i taints
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
[ K8s - Control pod-probe-status ]: kubectl describe node node1 | grep -i taints
Taints:             Env=dev:NoSchedule
[ K8s - Control pod-probe-status ]: kubectl describe node node2 | grep Taint
Taints:             gpu=true:NoSchedule
```





### 容忍度

容忍度是给予 Pod 的描述，容忍度的作用在于允许 Pod 调度到有污点的 node 上。



描述

```bash
tolerations
```

路径

```yaml
spec:
  tolerations:
```

查询

```bash
kubectl explain pod.spec.tolerations
```

值

```yaml
spec:
  tolerations:
  - key:          # 污点键
    operator:     # 污点值
    value:        # 操作符
    effect:       # 匹配污点容忍度
    
关于污点容忍度

NoExecute
NoSchedule
PreferNoSchedule

# Pod 中 tolerations 字段里的 effect，必须和目标 Node 上的污点的 effect 完全匹配，才能容忍该污点。
```



### 实例



将 node2 添加污点

```bash
[ K8s - Control ~ ]: kubectl taint nodes node2 gpu=true:NoSchedule
node/node2 tainted
```

创建 yaml 文件使 Pod 容忍该污点

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-deploy
  namespace: default
  labels:
    app: myapp
    Occu: Archori
    release: canary
spec: 
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
    
# 关于操作符

Equal     # 容忍度键必须和容忍度值相等
Exists    # 不关心容忍度值。容忍度键存在即可，此时容忍度值为空

# 更多的

当 effect 为空字符串时，Pod 调度可以容忍任何容忍度类型的节点
```

检查

```bash
[ K8s - Control pod-1 ]: kubectl apply -f taint.yaml 
pod/myapp-deploy created
[ K8s - Control pod-1 ]: kubectl get pods
NAME           READY   STATUS              RESTARTS   AGE
myapp-deploy   0/1     ContainerCreating   0          6s
```



## LifeCycle



### 初始化容器

在 Kubernetes 中，initContainer（初始化容器）是专门用于在主应用容器启动**之前**执行某些初始化任务的容器。它们和普通的 container 容器一样写在 Pod 的配置中，但有如下几个核心特点和用途：

1. **顺序执行**：多个 initContainer 会按顺序依次执行，**只有全部成功结束后**，主容器 (containers) 才会启动。
2. **失败重试**：如果任意一个 initContainer 失败，Kubernetes 会不断重启 Pod，直到它们成功完成或超出重试限制。
3. **临时运行**：initContainer 运行完就退出，不会在 Pod 生命周期中常驻

| 等待资源就绪 | 比如等待数据库或某个依赖服务可用（可以用 wget / curl 轮询）。 |
| ------------ | ------------------------------------------------------------ |
| 初始化配置   | 下载配置文件、初始化目录结构或从远程加载密钥。               |
| 设置权限     | 比如使用 chown 修改某些目录权限，主容器可能没有权限这么做。  |
| 数据预处理   | 从 Git 拉取代码、准备模型文件、创建数据库 schema。           |
| 依赖注入     | 在运行主应用前，动态注入一些运行时依赖（如凭证、配置模板）。 |

描述

```bash
initContainers
```

路径

```yaml
spec:
  initContainers:
```

查询

```bash
kubectl explain pod.spec.initContainers
```

值

```yaml
spec:
  initContainers:
  - name:
  - name: 
```



### 主容器

主容器是生产容器。



#### 容器钩子

当初始化容器运行结束后，Kubernetes 允许用户定义 **容器启动后钩子 (post start hook)** 和 **容器结束前钩子 (pre stop hook)** 用于检测运行状态和释放内存，优雅结束程序。

**postStart : ** 该钩子在主容器创建后立即执行，若钩子执行的探测任务失败，则 Kubernetes 会立刻杀死容器，并根据用户定义的重启策略选择是否重启该容器。该钩子不需要传递任何参数。

**preStop : ** 该钩子在容器结束前执行，用于释放内存以及结束程序。

容器钩子在 **生命周期 (lifecycle)** 下

描述

```bash
lifecycle
```

路径

```yaml
spec:
  containers:
    lifecycle:
```

查询

```bash
kubectl explain pod.spec.containers.lifecycle
```



##### 启动后钩子

描述

```bash
postStart
```

路径

```yaml
spec:
  containers:
    lifecycle:
      postStart:
```

查询

```bash
kubectl explain pod.spec.containers.lifecycle.postStart
```



##### 结束前钩子

描述

```bash
preStop
```

路径

```yaml
spec:
  containers:
    lifecycle:
      preStop:
```

查询

```bash
kubectl explain pod.spec.containers.lifecycle.preStop
```



例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: life-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    lifecycle:              # 生命周期字段
      postStart:            # 启动后钩子
        exec:               # 参数
          command: ["/bin/sh","-c","echo 'lifecycle hookshandler' > /usr/share/nginx/html/test.html"]
      preStop:              # 停止前钩子
        exec:               # 参数
          command:
          - "/bin/sh"       # 使用 sh 解释器
          - "-c"            # 参数
          - "nginx -s stop" # 终止 nginx 服务
          
# 注： sh -c 命令用于启动一个 sh 解释器并且传入参数
sh -c <COMMAND>
# 而 sh 则是启动一个使用 sh 解释器的交互式 shell
```



## Container Probe

Kubernetes 中的“容器探测”（Probe）是一种由 kubelet 定期执行的健康检查机制，用来判断 Pod 中的容器是否处于可用和健康的状态。主要包括三种类型

### 启动探测

**startusProbe** 探针用于探测 Pod 内容器是否启动。如果启动了启动探测，则其他两种探针都将失效。如果探针失败，kubelet 将杀死容器，并根据重启策略重新启动容器。若容器没有提供启动探测，则默认状态为 Success 。

#### exec 模式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startupprobe
spec:
  containers:
  - name: startup
    image: xianchao/tomcat-8.5-jre8:v1
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8080
    startupProbe:                # 启动探测
      exec:                      # Handler
        command:
        - "/bin/sh"
        - "-c"
        - "ps aux | grep tomcat"
      initialDelaySeconds: 20    # 容器启动后多久开始探测
      periodSeconds: 20          # 执行探测时间间隔
      timeoutSeconds: 10         # 响应超时时间
      successThreshold: 1        # 有效成功次数
      failureThreshold: 3        # 有效失败次数
```



#### tcpSocket 模式

描述

```bash
tcpSocket
```

路径

```yaml
spec:
  containers:
  - startupProbe:
      tcpScoket:
```

查询

```bash
kubectl explain pod.spec.containers.startupProbe.tcpSocket
```

字段

```yaml
host  # 链接主机名，默认 pod IP
port  # 检测端口是否存在，必须字段
```



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startupprobe
spec:
  containers:
  - name: startup
    image: xianchao/tomcat-8.5-jre8:v1
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8080
    startupProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 20
      periodSeconds: 20
      timeoutSeconds: 10
      successThreshold: 1
      failureThreshold: 3
```



#### httpGet 模式



描述

```bash
httpGet
```

路径

```yaml
spec:
  containers:
    startupProbe:
      httpGet:
```

查询

```bash
kubectl explain pod.spec.containers.startupProbe.httpGet
```

字段

```yaml
path: 
port: 
host:
scheme:
httpHeaders:
```

例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startupprobe
spec:
  containenrs:
  - name: startup
    image: xianchao/tomcat-8.5-jre8:v1
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /
        port: 8080
      initialDelaySeconds: 20
      periodSeconds: 20
      successThreshold: 1
      failureThreshold: 3
```



> [!NOTE]
>
> **exec** 模式在于检测容器内执行命令是否成功，自由度高
>
> **tcpSocket** 模式在于提供一个模块，检测特定主机名和端口
>
> **httpGet** 模式在于发送一个 http/https 请求，检查是否成功



### 存活探测

**livenessProbe** 探针用于探测 Pod 内容器是否存活。如果测试失败，则认为容器不健康。kubelet 将根据 restartPolicy 策略重启 Pod 。如果容器配置中没有配置该探针，则 kubelet 将认为存活探测一直为 Success 状态。



#### exec 模式

描述

```bash
exec
```

路径

```yaml
spec:
  containers:
    livenessProbe:
      exec:
```

查询

```bash
kubectl explain pod.spec.containers.livenessProbe.exec
```

字段

```yaml
command: 
```

例子

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec
  labels:
    app: liveness
spec:
  containers:
  - name: liveness
    image: busybox:1.28
    imagePullPolicy: IfNotPresent
    args:            # 容器启动时执行的命令
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      initialDelaySeconds: 10   # 延迟检测时间
      periodSeconds: 5          # 检测时间间隔
      exec:
        command:
        - cat
        - /tmp/healthy
```

#### httpGet 模式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
  labels:
    test: liveness
spec: 
  containers:
  - name: liveness
    image: mydlqclub/springboot-helloworld:0.0.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      initialDelaySeconds: 20
      periodSeconds: 5
      timeoutSeconds: 10
      httpGet:
        scheme: HTTP
        port: 8081
        path: /actuator/health
```

#### tcpSocket 模式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp
  labels:
    app: liveness
spec: 
  containers:
  - name: liveness
    image: docker.io/xianchao/nginx:v1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      initialDelaySeconds: 15
      periodSeconds: 20
      tcpSocket:
        port: 80
```





### 就绪探测

**readinessProbe** 探针用于探测容器内应用是否可以接受请求。当探测成功后容器将被标记为就绪状态，并提供对外网络访问。若探测失败，则 kubelet 将标记容器为为就绪状态，并从前端负载移除。



```yaml
apiVersion: v1
kind: Service
metadata:
  name: springboot
  labels:
    app: springboot
spec:
  type: NodePort
  ports:
  - name: server
    port: 8080
    targetPort: 8080
    nodePort: 31180
  - name: management
    port: 8081
    targetPort: 8081
    nodePort: 31181
  selector:
    app: springboot
---
apiVersion: v1
kind: Pod
metadata:
  name: springboot
  labels:
    app: springboot
spec:
  containers:
  - name: springboot
    image: mydlqclub/springboot-helloworld:0.0.1
    imagePullPolicy: IfNotPresent
    ports:
    - name: server
      containerPort: 8080
    - name: management
      containerPort: 8081
    readinessProbe:
      initialDelaySeconds: 20   
      periodSeconds: 5          
      timeoutSeconds: 10   
      httpGet:
        scheme: HTTP
        port: 8081
        path: /actuator/health

```







### 探针 Handler

每种 Probe 都可以通过以下三种方式之一来实现探测逻辑：

**exec**：在容器内执行命令，返回状态码 0 视为成功。

**httpGet**：向容器内的某个 HTTP 端点发起 GET 请求，返回 2xx 或 3xx 视为成功。

**tcpSocket**：尝试与容器的某个 TCP 端口建立连接，连接成功即视为成功。



### 探针字段



描述

```bash
periodSeconds
```

路径

```yaml
spec:
  containers:
    livenessProbe:
      periodSenconds:
```

查询

```bash
kubectl explain pod.spec.containers.livenessProbe.periodSeconds
```



## Controller

简而言之，控制器代理我们控制集群调度。



### ReplicaSet

**ReplicaSet** 是 **Kubernetes** 中的一个副本控制器，简称 **rs** ，主要作用是控制其管理的 Pod ，将副本数量维持在预设的个数，保证 Pod 在集群中正常运行。是一种基础副本控制器。

#### 关键字段

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name:            # 指定 ReplicaSet 资源名称
  namespace:       # 指定 ReplicaSet 资源命名空间
  labels:          # 指定 ReplicaSet 资源标签
spec: 
  replicas:        # 指定的副本数量
  selector:        # 标签选择器 --必须--
  minReadySeconds: # Ready 状态持续多少秒才视为 Available
  matchLabels:     # 设置链接模版 Pod 标签-----+
  template:        # 指定模版                 |
    metadata:        # 指定创建 Pod 的元数据   |
      labels:          # 设置 Pod 具有的标签---+
    spec:
      containers:
      - name:
        images:
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  namespace: default
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier1: frontend1
  template:
    metadata: 
      labels: 
        tier1: frontend1
    spec:
      containers:
      - name: php-redis
        image: docker.io/yecc/gcr.io-google_samples-gb-frontend:v3
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        startupProbe:
           periodSeconds: 5
           initialDelaySeconds: 20
           timeoutSeconds: 10
           httpGet:
             scheme: HTTP
             port: 80
             path: /
        livenessProbe:
           periodSeconds: 5
           initialDelaySeconds: 20
           timeoutSeconds: 10
           httpGet:
             scheme: HTTP
             port: 80
             path: /
        readinessProbe:
           periodSeconds: 5
           initialDelaySeconds: 20
           timeoutSeconds: 10
           httpGet:
             scheme: HTTP
             port: 80
             path: /
```

```yaml
[ K8s - Control rs ]: kubectl apply -f replicaset.yaml 
replicaset.apps/frontend created
[ K8s - Control rs ]: kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       29s
[ K8s - Control rs ]: kubectl delete -f replicaset.yaml 
replicaset.apps "frontend" deleted
```



#### 节点扩容

只需要修改 rs 配置文件再 apply 即可

```yaml
...
  replicas: 3 # replicas : 4
...
```

```yaml
[ K8s - Control rs ]: kubectl apply -f replicaset.yaml 
replicaset.apps/frontend configured
[ K8s - Control rs ]: kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-76465   1/1     Running   0          2m34s
frontend-pnjzg   1/1     Running   0          2m34s
frontend-qw6mj   1/1     Running   0          2m34s
frontend-rrsxg   0/1     Running   0          10s
```



#### 节点缩容

```yaml
...
  replicas: 4 # replicas : 2
...
```

```yaml
[ K8s - Control rs ]: kubectl apply -f replicaset.yaml 
replicaset.apps/frontend configured
[ K8s - Control rs ]: kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
frontend-pnjzg   1/1     Running   0          3m59s
frontend-qw6mj   1/1     Running   0          3m59s
```



#### 镜像升级

当使用更新镜像替换时，直接在 rs 配置文件中替换，使用 apply 。此时，容器内镜像不会动态更新，若手动删除 Pod ，新创建的镜像将使用新的镜像。（ ReplicaSet 资源无法实现滚动更新 ）



### Deployment

在生产环境中，常常遇到镜像升级，当使用 ReplicaSet 资源时，必须人工删除 Pod 使其更新镜像。但在实际生产环境中，常使用“蓝绿发布”，即原先有一个 rs1 ，再创建一个 rs2 ，通过修改 service 标签，匹配到 rs2 控制器，完成镜像升级。于是，**Deployment** 控制器应运而生。

Deployment 控制器是建立在 rs 之上的一个控制器，可以管理多个 rs ，每次更新镜像版本，都会生成一个新的 rs，把旧的 rs 替换掉。此时，多个 rs 同时存在，但只有一个 rs 运行。

#### 工作原理

Deployment 可以通过声明式定义，直接在命令行通过命令方式完成对应资源版本的修改，也就是通过打补丁的方式进行修改。



#### 关键字段

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
spec:
  replicas:                # 副本数
  minReadySeconds:         # Ready 状态持续多少秒才视为 Available
  paused:                  # 是否暂停更新 true/false
  progressdeadlineSeconds: # 超时失败时间
  revisionHistoryLimit:    # 历史版本保留数（回滚用）
  strategy:                # 更新策略
    type: RollingUpdate/Recreate # 更新类型
    rollingUpdate:           # 仅在 RollingUpdate 启用
      maxSurge:                # 滚动更新允许额外创建 Pod 的数量（整数或百分比）向上补齐
      maxUnabailable:          # 滚动更新时允许不可用的最大 Pod 数量（整数或百分比）向下补齐
  selector:                # 标签选择器 --必须-- 定义哪些 Pod 属于该 Deployment 必须匹配 .spec.template.metadata.labels
  template:                # 模版
  
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: docker.io/janakiramm/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        startupProbe:
          periodSeconds: 5
          initialDelaySeconds: 20
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
        livenessProbe:
          periodSeconds: 5
          initialDelaySeconds: 20
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
        readinessProbe:
          periodSeconds: 5
          initialDelaySeconds: 20
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
```

```bash
[ K8s - Control deployment ]: kubectl apply -f deploy-demo.yaml 
deployment.apps/myapp-v1 created
[ K8s - Control deployment ]: kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
myapp-v1   2/2     2            2           98s
[ K8s - Control deployment ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-7b4647c584   2         2         2       103s
[ K8s - Control deployment ]: kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
myapp-v1-7b4647c584-l6j6c   1/1     Running   0          107s
myapp-v1-7b4647c584-rkjmk   1/1     Running   0          107swa
```



#### 节点扩容

直接修改配置文件 replica 字段后 apply 即可

#### 节点缩容

直接修改配置文件 replica 字段后 apply 即可



#### 滚动更新

```bash
[ K8s - Control ~ ]: kubectl get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
myapp-v1-7b4647c584-l6j6c   1/1     Running   0          166m
myapp-v1-7b4647c584-llztl   0/1     Running   0          15s
myapp-v1-7b4647c584-rkjmk   1/1     Running   0          166m
############################
myapp-v1-7b4647c584-llztl   0/1     Running   0          25s
myapp-v1-7b4647c584-llztl   1/1     Running   0          25s
myapp-v1-8479994f8c-m66jd   0/1     Pending   0          0s
myapp-v1-8479994f8c-m66jd   0/1     Pending   0          0s
myapp-v1-8479994f8c-m66jd   0/1     ContainerCreating   0          0s
myapp-v1-8479994f8c-m66jd   0/1     ContainerCreating   0          1s
myapp-v1-8479994f8c-m66jd   0/1     Running             0          2s
myapp-v1-8479994f8c-m66jd   0/1     Running             0          25s
myapp-v1-8479994f8c-m66jd   1/1     Running             0          25s
myapp-v1-7b4647c584-llztl   1/1     Terminating         0          60s
myapp-v1-8479994f8c-pv5zw   0/1     Pending             0          1s
myapp-v1-8479994f8c-pv5zw   0/1     Pending             0          1s
myapp-v1-8479994f8c-pv5zw   0/1     ContainerCreating   0          1s
myapp-v1-7b4647c584-llztl   1/1     Terminating         0          61s
myapp-v1-7b4647c584-llztl   0/1     Completed           0          61s
myapp-v1-8479994f8c-pv5zw   0/1     ContainerCreating   0          1s
myapp-v1-8479994f8c-pv5zw   0/1     Running             0          2s
myapp-v1-7b4647c584-llztl   0/1     Completed           0          62s
myapp-v1-7b4647c584-llztl   0/1     Completed           0          62s
myapp-v1-8479994f8c-pv5zw   0/1     Running             0          26s
myapp-v1-8479994f8c-pv5zw   1/1     Running             0          26s
myapp-v1-7b4647c584-rkjmk   1/1     Terminating         0          167m
myapp-v1-8479994f8c-6vlvl   0/1     Pending             0          0s
myapp-v1-8479994f8c-6vlvl   0/1     Pending             0          0s
myapp-v1-8479994f8c-6vlvl   0/1     ContainerCreating   0          0s
myapp-v1-7b4647c584-rkjmk   1/1     Terminating         0          167m
myapp-v1-7b4647c584-rkjmk   0/1     Completed           0          167m
myapp-v1-8479994f8c-6vlvl   0/1     ContainerCreating   0          1s
myapp-v1-7b4647c584-rkjmk   0/1     Completed           0          167m
myapp-v1-7b4647c584-rkjmk   0/1     Completed           0          167m
myapp-v1-8479994f8c-6vlvl   0/1     Running             0          2s
myapp-v1-8479994f8c-6vlvl   0/1     Running             0          26s
myapp-v1-8479994f8c-6vlvl   1/1     Running             0          26s
myapp-v1-7b4647c584-l6j6c   1/1     Terminating         0          168m
myapp-v1-7b4647c584-l6j6c   1/1     Terminating         0          168m
myapp-v1-7b4647c584-l6j6c   0/1     Completed           0          168m
myapp-v1-7b4647c584-l6j6c   0/1     Completed           0          168m
myapp-v1-7b4647c584-l6j6c   0/1     Completed           0          168m
^C[ K8s - Control ~ ]: kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
myapp-v1-8479994f8c-6vlvl   1/1     Running   0          52s
myapp-v1-8479994f8c-m66jd   1/1     Running   0          103s
myapp-v1-8479994f8c-pv5zw   1/1     Running   0          78s
```

```bash
[ K8s - Control deployment ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-7b4647c584   3         3         3       167m
myapp-v1-8479994f8c   1         1         0       16s
[ K8s - Control deployment ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-7b4647c584   2         2         2       167m
myapp-v1-8479994f8c   2         2         1       35s
[ K8s - Control deployment ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-7b4647c584   1         1         1       167m
myapp-v1-8479994f8c   3         3         2       57s
[ K8s - Control deployment ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
myapp-v1-7b4647c584   0         0         0       168m
myapp-v1-8479994f8c   3         3         3       88s
```



#### 版本回退

```bash
kubectl rollout undo deployment/myapp-v1 --to-revision=<Deploy_Vision> -n <NameSpace_Name>
```



#### 滚动更新策略

通过调整 **maxSurge** 与 **maxUnavailable** 字段的值控制滚动更新策略

**maxSurge** 控制更新时每一阶段更新多少数量的 Pod ，越大更新越快，响应瞬间资源占用也越高

**maxUnavailable** 控制更新时最少可用 Pod 数量，一般设置 0



描述

```bash
rollingUpdate
```

路径

```yaml
spec:
  strategy:
    rollingUpdate:
```

查询

```bash
kubectl explain deployment.spec.strategy.rollingUpdate
```

字段

```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1          #
      maxUnavailable: 1    #
```



#### 蓝绿部署

Kubernetes 本身不支持蓝绿部署，但通过用户的规划，创建新的 deployment，更新 service 指向新的 deployment 资源。

#### 金丝雀发布

金丝雀发布（又称灰度发布）指更新先发布一台/一部分，做流量验证。



### StatefulSet

**StatefulSet（ 有状态服务控制器 ）**为每个 Pod 提供持久化存储目录。



#### 关键字段

```yaml
apiVersion: v1
kind: Service
metadata:
  name:
  labels:          # Service 资源标签
spec:
  ports:
  - name:          # 端口名称
    port:          # Service 资源端口（入口）
  clusterIP:       # 指定 Service 资源 IP
  selector:
    matchLabels:   # 指定 Pod 标签连接 Pod
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name:            # StatefulSet 资源名称
spec:
  replicas:        # 副本数
  selector:        #             -- <required> --
    matchLabels:   # 连接 Pod
  serviceName:     # 连接四层代理  -- <required> --
  template:
    metadata:
      labels:      # 被连接 Pod 标签
    spec:
      containers:
      - name:
        image:
        imagePullPolicy:
        ports:
        - name:    # 端口命名
          containerPort:
        volumeMounts:
        - name:      # 连接卷名称
          mountPath: # 卷挂载路径
  volumeClaimTemplates:    # 卷申请模版 自动申请PV/PVC
  - metadata:
      name:                # 卷名称
    spec:
      accessModes:         # 权限
      storageClassName:    # 连接存储类
      resources:
        requests:
          storage:         # 申请容量
```

例

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - name: web
    port: 80
  clusterIP: None
  selector:
    matchLabels:
      app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
  serviceName: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - name: web
          containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: nfs
      resources:
        requests:
          storage: 1Gi
```

> [!NOTE]
>
> StatefulSet 管理的 Pod 名称是有序的，每个 Pod 都有独自的域名，即
>
> ```bash
> <PodName>.<Service_Name>.<Service_NameSpace>.svc.cluster.local
> ```
>
> StatefulSet 由 Service 四层代理驱动



#### 更新策略

描述

```bash
updateStrategy
```

路径

```yaml
spec:
  updateStrategy:
```

查询

```bash
kubectl explain sts.spec.updateStrategy
```

字段

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
spec:
  updateStrategy:     # 更新策略
    rollingUpdate:      # 滚动更新
      partition:          # 更新序号大约等于值的 Pod
      maxUnavailable:     # 最大失效 Pod 数
```



### DaemonSet

**DaemonSet** 控制器能确保 Kubernetes 集群中所有节点都运行一个相同副本，当 Kubernetes 集群中新增 node 节点时，DaemonSet 控制器会在新增节点中创建一个相同副本，相反，当节点在集群中被移除，相应的副本也将销毁。

工作原理：

**DaemonSet** 控制器会监听 Kubernetes 的 daemonset 对象，pod对象，node对象，若对象发生变动，就会触发 syncLoop 循环让 Kubernetes 集群朝着 daemonset 对象描述的状态进行演进。

#### 关键字段

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: 
  labels:
spec:
  selector:
  template:
    metadata:
      labels:
    spec:
      containers:
      - name:
        image:
        imagePullPolicy:
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata: 
  labels:
    k8s-app: fluentd-logging
  name: fluentd-elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
     name: fluentd-elasticsearch
  template:
    metadata:
     name: fluentd
     labels:
       name: fluentd-elasticsearch
    spec:
     tolerations:
     - key: node-role.kubernetes.io/master
       effect: NoSchedule
     containers:
     - name:  fluentd-elasticsearch
       image: xianchao/fluentd:v2.5.1
       resources:
         limits: 
           memory: 500Mi
         requests: 
           cpu: 100m
           memory: 200Mi
       volumeMounts:
       - name: varlog
         mountPath: /var/log
       - name: varlibdockercontainers
         mountPath: /var/lib/docker/containers
         readOnly: true
     terminationGracePeriodSeconds: 30
     volumes:
     - name: varlog
       hostPath:
          path: /var/log
     - name: varlibdockercontainers
       hostPath:
          path: /var/lib/docker/containers
```

```bash
[ K8s - Control daemonset ]: kubectl get pods -n kube-system -l name=fluentd-elasticsearch -owide
NAME                          READY   STATUS    RESTARTS   AGE    IP               NODE    NOMINATED NODE   READINESS GATES
fluentd-elasticsearch-lkx8f   1/1     Running   0          3m6s   10.244.166.152   node1   <none>           <none>
fluentd-elasticsearch-w9hvw   1/1     Running   0          3m6s   10.244.104.26    node2   <none>           <none>
```



#### 更新策略

与 Deployment 更新策略相同。



## Service

在 Kubernetes 中，Pod 具有生命周期，在重启后 Pod IP 会发生变化，若服务关联 IP ，则重启后服务会丢失计算资源，为了解决这个问题，Kubernetes 定义了 Service 资源，它是一个服务访问入口，代理客户端流量访问计算集群，又称 **四层负载均衡**。

Kubernetes 在创建 Service 时，会根据标签选择器查找 Pod ，据此创建与 Service 同名的 endpoint 对象，当 Pod 地址发生变化时，endpoint 也随之发生变化，service 接受前端 client 请求的时候，通过 endpoint，找到转发到哪个 Pod 进行访问的地址。

### 关键字段

```yaml
apiVersion: v1
kind: Service
metadata:
  name:                # Service 资源名称
  labels:              # Service 资源标签
spec:
  type:                # Service 资源类型
  ports:               # 端口设置
  - port:                # Service 资源端口（入口）
    protocol:            # 网络协议 TCP/UDP/SCTP
    targetPort:          # 对容器内转发到的端口
  selector:            # 标签选择器（连接到匹配条件容器）
```



### ClusterIp

创建基本环境

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        startupProbe:
          periodSeconds: 5
          initialDelaySeconds: 60
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
        livenessProbe:
          periodSeconds: 5
          initialDelaySeconds: 60
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
        readinessProbe:
          periodSeconds: 5
          initialDelaySeconds: 60
          timeoutSeconds: 10
          httpGet:
            scheme: HTTP
            port: 80
            path: /
```

```bash
[ K8s - Control service ]: kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-6948b4fb8b-f28qw   1/1     Running   0          70s
my-nginx-6948b4fb8b-mb26z   1/1     Running   0          70s
[ K8s - Control service ]: kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
my-nginx-6948b4fb8b   2         2         2       74s
[ K8s - Control service ]: kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx   2/2     2            2           77s
```

创建 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec: 
  type: ClusterIP
  ports:
  - port: 80        # 对外端口（流量进入端口）
    protocol: TCP
    targetPort: 80  # 对内端口（流量转发端口）
  selector:
    run: my-nginx
```

```bash
[ K8s - Control service ]: kubectl apply -f service_test.yaml 
service/my-nginx created
[ K8s - Control service ]: kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   190d
my-nginx     ClusterIP   10.98.137.35   <none>        80/TCP    8s
```

```bash
[ K8s - Control service ]: kubectl describe svc my-nginx
Name:                     my-nginx
Namespace:                default
Labels:                   run=my-nginx
Annotations:              <none>
Selector:                 run=my-nginx
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.98.137.35
IPs:                      10.98.137.35
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                10.244.166.149:80,10.244.104.18:80
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

```bash
# 自动存入 endpoint 对象
[ K8s - Control service ]: kubectl get ep
NAME         ENDPOINTS                            AGE
kubernetes   192.168.152.100:6443                 190d
my-nginx     10.244.104.18:80,10.244.166.149:80   2m30s
```

```bash
# 在节点上查询网络信息
[ K8s - Node 1 ~ ]: ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.96.0.1:443 rr
  -> 192.168.152.100:6443         Masq    1      4          0         
TCP  10.96.0.10:53 rr
  -> 10.244.42.199:53             Masq    1      0          0         
  -> 10.244.42.200:53             Masq    1      0          0         
TCP  10.96.0.10:9153 rr
  -> 10.244.42.199:9153           Masq    1      0          0         
  -> 10.244.42.200:9153           Masq    1      0          0         
TCP  10.98.137.35:80 rr
  -> 10.244.104.18:80             Masq    1      0          0         
  -> 10.244.166.149:80            Masq    1      0          0         
UDP  10.96.0.10:53 rr
  -> 10.244.42.199:53             Masq    1      0          0         
  -> 10.244.42.200:53             Masq    1      0          0         
```

> [!NOTE]
>
> 这种模式的缺点在于无法将 Pod 置于外网环境，只能在 Kubernetes 集群节点间访问。



### NodePort

创建基本环境

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx-nodeport
spec:
  selector:
    matchLabels:
      run: my-nginx-nodeport
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx-nodeport
    spec:
      containers:
      - name: my-nginx-nodeport-container
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```

```bash
[ K8s - Control service ]: kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
my-nginx-nodeport-84c5c6d5c4-cv8xj   1/1     Running   0          7s
my-nginx-nodeport-84c5c6d5c4-gm78w   1/1     Running   0          7s
[ K8s - Control service ]: kubectl get rs
NAME                           DESIRED   CURRENT   READY   AGE
my-nginx-nodeport-84c5c6d5c4   2         2         2       16s
[ K8s - Control service ]: kubectl get deploy
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
my-nginx-nodeport   2/2     2            2           27s
```

创建 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-nodeport
  labels:
    run: my-nginx-nodeport
spec: 
  type: NodePort
  ports:
  - port: 80          # 集群内暴露的端口
    protocol: TCP
    targetPort: 80    # 转发给计算集群的端口号
    nodePort: 30380   # 对公网暴露的端口
  selector:
    run: my-nginx-nodeport
```

网络结构

```bash
[集群外部用户] 
        |
        | 访问 NodeIP:30380
        v
[Kubernetes Node 上的 NodePort]
        |
        | 访问 port:80（Service 层）
        v
[Service 根据 selector 选择 Pod]
        |
        | 转发到 targetPort:80（Pod 内容器端口）
        v
[运行 Nginx 的 Pod]
```

```bash
[ K8s - Control service ]: kubectl apply -f service_nodeport.yaml 
service/my-nginx-nodeport created
[ K8s - Control service ]: kubectl get svc
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP        190d
my-nginx-nodeport   NodePort    10.97.206.119   <none>        80:30380/TCP   9s
```



### ExternalName

**跨名称空间访问** ，例如 default 名称空间下的 client 服务想要访问 nginx-ns 名称空间下的 nginx-svc 服务。

```bash
[ K8s - Control service ]: kubectl create ns nginx-ns
namespace/nginx-ns created
```

创建控制器

```yaml
# server_nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: nginx-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec: 
      containers:
      - name: nginx
        image: nginx
        imagePullPolicy: IfNotPresent
```

```bash
[ K8s - Control service ]: kubectl apply -f server_nginx.yaml 
deployment.apps/nginx created
[ K8s - Control service ]: kubectl get deployment -n nginx-ns
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           22s
```

创建 Service

```yaml
# nginx_svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: nginx-ns
spec:
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 80
```

```bash
[ K8s - Control service ]: kubectl apply -f nginx_svc.yaml 
service/nginx-svc created
[ K8s - Control service ]: kubectl get svc -n nginx-ns
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-svc   ClusterIP   10.109.143.72   <none>        80/TCP    17s
```

在 default 名称空间下创建 Pod 作为客户端

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: busybox
        imagePullPolicy: IfNotPresent
        command: ["/bin/sh","-c","sleep 36000"]
```

```bash
[ K8s - Control service ]: kubectl apply -f client.yaml 
deployment.apps/client created
[ K8s - Control service ]: kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
client-5fc5f78764-pjkns   1/1     Running   0          5s
```

创建客户端 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-svc
spec:
  type: ExternalName
  externalName: nginx-svc.nginx-ns.svc.cluster.local
  ports:
  - name: http
    port: 80
    targetPort: 80
```

```bash
# 其中 externalName 是资源的完全合规域名，格式为：
<Service_Name>.<NameSpace_Name>.svc.cluster.local
```

```bash
[ K8s - Control service ]: kubectl apply -f client_svc.yaml 
service/client-svc created
[ K8s - Control service ]: kubectl get svc -n default
NAME         TYPE           CLUSTER-IP   EXTERNAL-IP                            PORT(S)   AGE
client-svc   ExternalName   <none>       nginx-svc.nginx-ns.svc.cluster.local   80/TCP    12s
kubernetes   ClusterIP      10.96.0.1    <none>                                 443/TCP   190d
# 可以看到，client 的 Service 资源没有IP，而是链接到了 nginx-svc.nginx-ns.svc.cluster.local
```

进入 default 名称空间下的客户端 Pod

```bash
[ K8s - Control service ]: kubectl exec -it client-5fc5f78764-pjkns -- /bin/sh
```

直接访问客户端 Pod 上的 Service 资源即可自动连接到另一名称空间下的资源

```bash
/  wget -q -O - client-svc
<!DOCTYPE html>
<html>
<head>
...
...

/  ping client-svc
PING client-svc (10.109.143.72): 56 data bytes
# 直接 ping 也是指向的另一 namespace 下的 Pod
# 这些都由 Service 直接代理
[ K8s - Control ~ ]: kubectl get svc -n nginx-ns
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-svc   ClusterIP   10.109.143.72   <none>        80/TCP    26m
```



### Endpoint

在 Kubernetes 中，Endpoint 资源是一个表示一组 IP 地址和端口的对象，它将 Kubernetes Service 与实际提供服务的 Pod 关联起来，简单说，它是 Service 地址簿。

Endpoint 是一种 Kubernetes API 资源，用于记录某个 Service 选中的后端 Pod 的 IP 和端口信息。



例

创建 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: ClusterIP
  ports:
  - port: 80
  
# 无 selector 则不会关联 Pod 则不会创建 endpoint
```

```bash
[ K8s - Control endpoint ]: kubectl describe svc nginx
Name:                     nginx
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 <none>
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.101.248.246
IPs:                      10.101.248.246
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                <none>
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>
```

创建 Endpoints

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: nginx
subsets:
- addresses:
  - ip: 192.168.152.102  # 这里是关联主机的IP地址
  ports:
  - port: 80             # 关联主机提供服务的端口
```

```bash
[ K8s - Control endpoint ]: kubectl apply -f nginx.endpoint.yaml 
endpoints/nginx created
[ K8s - Control endpoint ]: kubectl get ep
NAME         ENDPOINTS              AGE
kubernetes   192.168.152.100:6443   190d
nginx        192.168.152.102:80     5s
```



### coredns

启动 Pod

```yaml
aprVersion: v1
kind: Pod
metadata:
  name: dig
  namespace: default
spec:
  containers:
  - name: dig
    image: xianchao/dig:latest
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
```

```bash
[ K8s - Control coredns ]: kubectl apply -f dig.yaml 
pod/dig created
[ K8s - Control coredns ]: kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
dig    1/1     Running   0          6s
```

```bash
[ K8s - Control coredns ]: kubectl exec -it dig -- /bin/sh
/ # nslookup kubernetes
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	kubernetes.default.svc.cluster.local
Address: 10.96.0.1
```

其实一般情况没啥用，它随着 kubeadm 安装所启动，coredns 正常运行才能保证容器之间可以以 FQDN（完全合规域名）访问。



## Storage

由于 Pod 具有生命周期，当 Pod 销毁后，相应数据也被销毁，为了保存数据，Kubernetes 引用了 **Persistent Storage 数据持久化 **技术。



### emptyDir

是一种 Pod 生命周期内的临时目录，非持久化

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-empty
spec:
  containers:
  - name: container-empty
    image: nginx
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: chche-volume       # 容器内挂载生成的卷名称
      mountPath: /cache        # 挂载目录
  volumes:                     # 卷配置
  - emptyDir: {}               # 生成临时卷
    name: cache-volume         # 卷名称
```

```bash
[ K8s - Control storage ]: kubectl apply -f emptydir.yaml 
pod/pod-empty created
[ K8s - Control storage ]: kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
pod-empty   1/1     Running   0          6s
[ K8s - Control storage ]: kubectl get pods -owide
NAME        READY   STATUS    RESTARTS   AGE   IP              NODE    NOMINATED NODE   READINESS GATES
pod-empty   1/1     Running   0          13s   10.244.104.22   node2   <none>           <none>
```

临时目录存放在

```bash
/var/lib/kubelet/pods/<Pod_UID>/kubernetes.io~empty-dir/<Volume_Name>
```



### hostPath

直接使用宿主机文件系统中的目录或文件，用于测试或单节点集群

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-hostpath
spec:
  containers:
  - name: test-nginx
    image: nginx
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: test-volume
      mountPath: /test-nginx
  - name: test-tomcat
    image: docker.io/library/tomcat8.5-jre8-alpine
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: test-volume
      mountPath: /test-nginx
  volumes:
  - name: test-volume
    hostPath:
      path: /data1
      type: DirectoryOrCreate  # 若没有则创建 0755
```



### nfs

网络存储

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: nfs-test
spec:
  replicas: 3
  selector:
    matchLabels:
      storage: nfs
  template:
    metadata:
      labels:
        storage: nfs
    spec:
      containers:
      - name: test-nfs
        image: nginx
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - name: nfs-volumes
          mountPath: /usr/share/nginx/html # 挂载到
      volumes:
      - name: nfs-volumes         # 卷名
        nfs:                      # 卷类型
          server: 192.168.40.180  # nfs 服务端 IP
          path: /data/volumes     # nfs 提供卷路径
```



### PVC

**PersistentVolume (PV) **是集群中的一块存储，由管理员配置或使用存储类动态配置。他是集群中的资源，就像 Pod 是 k8s 集群资源一样，PV 是容量插件，如 Volumes，其生命周期独立于使用 PV 的任何单个 Pod

**PersistentVolumeClain (PVC) **是一个持久化存储卷，管理员可以在创建 Pod 时定义这个类型的存储卷，它类似一个 Pod。Pod 消耗节点资源，PVC 消耗 PV 资源。Pod 可以请求特定级别的资源，pvc 在申请 pv  的时候也可以请求特定大小和访问模式。

PV 是集群中的资源，PVC 是对这些资源的请求。

#### 关键字段

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: 
spec:
  nfs:
    server:
    path:
  accessModes:    # 访问模式
  capacity:
    storage:      # 容量
```

```bash
# 访问模式
["ReadWriteOnce"]     # 卷可以被一个节点以读写方式挂载
["ReadOnlyMany"]      # 卷可以被多个节点以只读方式挂载
["ReadWriteMany"]     # 卷可以被多个节点以读写方式挂载
["ReadWriteOncePod"]  # 卷可以被单个 Pod 以读写方式挂载
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-v1
spec:
  accessModes:   # 访问模式
  selector:
    matchLabels: # 标签选择器
    resources:
      requests:
        storage: # 请求容量
```



创建 PV

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: v1
  labels:
    app: v1
spec:
  nfs:
    server: 192.168.40.180
    path: /data/volume_test/v1
  accessModes: ["ReadWriteOnce"]
  capacity:
    storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: v2
  labels:
    app: v2
spec:
  nfs:
    server: 192.168.40.180
    path: /data/volume_test/v2
  accessModes: ["ReadOnlyMany"]
  capacity:
    storage: 2Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: v3
  labels:
    app: v3
spec:
  nfs:
    server: 192.168.40.180
    path: /data/volume_test/v3
  accessModes: ["ReadWriteMany"]
  capacity:
    storage: 3Gi
```

```bash
kubectl apply -f pv.yaml
```



创建 pvc

```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-v1     # pvc 名称
spec:
  accessModes: ["ReadWriteOnce"]
  selector:
    matchLabels:
      app: v1      # 匹配标签为 app=v1 的 pv
  resources:
    requests:
      storage: 1Gi # 请求容量
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-v1     # pvc 名称
spec:
  accessModes: ["ReadOnlyMany"]
  selector:
    matchLabels:
      app: v2      # 匹配标签为 app=v2 的 pv
  resources:
    requests: 
      storage: 2Gi # 请求容量
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-v1     # pvc 名称
spec:
  accessModes: ["ReadWriteMany"]
  selector:
    matchLabels:
      app: v3      # 匹配标签为 app=v3 的 pv
  resources:
    requests: 
      storage: 3Gi # 请求容量
```

```yaml
# 创建 Pod
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvc-test-1
spec:
  replicas: 3
  selector:
    matchLabels:
      cunchu: pvc
  template: 
    metadata:
      labels:
         cunchu: pvc
    spec:
      containers:
      - name: test-pvc
        image: xianchao/nginx:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - name: nginx-html
          mountPath: /usr/share/nginx/html
      volumes:
      - persistentVolumeClaim:
          claimName: pvc-v1
        name: nginx-html
```



## StorageClass

Kubernetes 提供一种自动创建 PV 的机制，叫 StorageClass，它的作用是创建 PV 模版，Kubernetes 集群管理员通过创建 StorageClass 资源可以动态生成一个存储卷 PV 供 PVC 使用。



### 关键字段

```yaml
apiVersion: v1
kind: StorageClass

```







创建控制器

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-provisioner
spec:
  selector:
    matchLabels:
       app: nfs-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-provisioner
    spec:
      serviceAccount: nfs-provisioner  # 认证授权
      containers:
        - name: nfs-provisioner
          image: registry.cn-beijing.aliyuncs.com/mydlq/nfs-subdir-external-provisioner:v4.0.0
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:                         # 环境变量
            - name: PROVISIONER_NAME
              value: example.com/nfs
            - name: NFS_SERVER
              value: 192.168.40.180
            - name: NFS_PATH
              value: /data/nfs_pro/
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.40.180
            path: /data/nfs_pro/
```

创建 PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-claim1
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs
```

创建 StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs
provisioner: example.com/nfs  # 供应商
```



## ConfigMap

**ConfigMap** 是 Kubernetes 中的资源对象，用于保存非机密性配置，数据可以用 key/value 键值对的形式保存，也可以通过文件保存。

由于在生产环境中，每个服务都有自己的配置文件，当节点扩容或配置发生变化时，须手动大批量修改配置文件，为改善这个问题，Kubernetes 引入了 ConfigMap 资源对象，它可以做成 volume 挂载到 Pod 中，实现统一的配置管理。

### 关键字段

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name:
  labels:
data:
  <File_Name>: |  # |代表多行
  <key>=<Value>
  ...
```

例

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
    app: mysql
data:
  master.cnf: |
    [mysqld]
    log-bin
    log_bin_trust_fuction_creators=1
    lower_case_table_names=1
  slave.cnf: |
    [mysqld]
    super-read-only
    log_bin_trust_function_creators=1
```

### 构建

#### 通过命令行

```bash
kubectl create configmap <cm_Name> --from-literal=<Key>=<Value> --from-literal=<Key>=<Value> ...
```

```bash
[ K8s - Control home ]: kubectl create cm cmtest --from-literal=home=/dir
configmap/cmtest created
[ K8s - Control home ]: kubectl get cm
NAME               DATA   AGE
cmtest             1      15s
kube-root-ca.crt   1      192d
[ K8s - Control home ]: kubectl describe cm cmtest
Name:         cmtest
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
home:
----
/dir


BinaryData
====

Events:  <none>
```



#### 通过文件

```bash
kubectl create configmap <cm_Name> --from-file=<Key>=<JSON_file> # 键为 <Key>
kubectl create configmap <cm_Name> --from-file=<JSON_file> # 键为 <JSON_file>
```

创建变量/配置文件

```bash
vim nginx_conf
```

```json
server{
  server_name www.nginx.com;
  listen 80;
  root /home/nginx/www/
}
```

```bash
[ K8s - Control cm ]: kubectl create cm www-nginx --from-file=www=./nginx.conf
configmap/www-nginx created
[ K8s - Control cm ]: kubectl describe cm www-nginx
Name:         www-nginx
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
www:           # 键
----
server{
  server_name www.nginx.com;
  listen 80;
  root /home/nginx/www/
}



BinaryData
====

Events:  <none>
```



#### 通过目录

```bash
kubectl create cm <cm_Name> --from-file=<Dir_path>
```

例

```bash
[ K8s - Control cm ]: mkdir test-a
[ K8s - Control test-a ]: cat ./*
server-id=1
server-id=2
```

```bash
[ K8s - Control test-a ]: kubectl create cm mysql-config --from-file=/home/cm/test-a/
configmap/mysql-config created
```

```bash
[ K8s - Control test-a ]: kubectl describe cm mysql-config
Name:         mysql-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
my-server.cnf:
----
server-id=1


my-slave.cnf:
----
server-id=2



BinaryData
====

Events:  <none>
```

> [!NOTE]
>
> 通过目录直接将目录下所有文件分别制作 Configmap 并直接以文件名命名键，文件内容命名值



### 挂载

#### 通过环境变量

##### configMapKeyRef

描述

```bash
configMapKeyRef
```

路径

```yaml
spec:
  containers:
    env:
      valueFrom:
        configMapKeyRef:
```

查询

```bash
kubectl explain pod.spec.containers.env.valueFrom.configMapKeyRef
```

创建 ConfigMap

```bash
[ K8s - Control cm ]: vim mysql-configmap.yaml
```

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
    app: mysql
data:
  log: "1"
  lower: "1"
```

```bash
[ K8s - Control cm ]: kubectl apply -f mysql-configmap.yaml 
configmap/mysql created
[ K8s - Control cm ]: kubectl get cm
NAME               DATA   AGE
cmtest             1      50m
kube-root-ca.crt   1      193d
mysql              2      5s
mysql-config       2      35m
www-nginx          1      42m
```

创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
  - name: mysql
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","sleep 3600"]
    env:
    - name: log_bin      # 创建环境变量 log_bin
      valueFrom:
        configMapKeyRef:
          name: mysql    # 指定 ConfigMap 名称
          key: log       # 指定 ConfigMap 中的 Key 值
    - name: lower        # 创建环境变量 lower
      valueFrom:
        configMapKeyRef:
          name: mysql
```

```bash
[ K8s - Control cm ]: kubectl apply -f mysql-Pod.yaml 
pod/mysql-pod created
[ K8s - Control cm ]: kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
mysql-pod   1/1     Running   0          5s
```

登录查看变量情况

```bash
[ K8s - Control cm ]: kubectl exec -it mysql-pod -c mysql -- /bin/sh
/  printenv
log_bin=1                 # log_bin
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=mysql-pod
SHLVL=1
HOME=/root
TERM=xterm
lower=1                   # lower
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```



##### envfrom

描述

```bash
envFrom
```

路径

```yaml
spec:
  containers:
  - envFrom:
    - configMapRef:
        name:         # 指定 ConfigMap 名称
```

查询

```bash
kubectl explain pod.spec.containers.envFrom
```



创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod-envfrom
spec:
  containers:
  - name: mysql
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh","-c","sleep 3600"]
    envFrom:
    - configMapRef:
        name: mysql
  restartPolicy: Never
```

```bash
[ K8s - Control cm ]: kubectl apply -f mysql-pod-envfrom.yaml 
pod/mysql-pod-envfrom created
[ K8s - Control cm ]: kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
mysql-pod           1/1     Running   0          14m
mysql-pod-envfrom   1/1     Running   0          5s
```

登录查询环境变量

```bash
[ K8s - Control cm ]: kubectl exec -it mysql-pod-envfrom -c mysql -- /bin/sh
/  printenv
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=mysql-pod-envfrom
SHLVL=1
HOME=/root
TERM=xterm
lower=1         # lower
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
log=1           # log
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
```



#### 通过卷

创建 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql
  labels:
    app: mysql
data:
  log: "1"
  lower: "1"
  my.cnf: |
    [mysqld]
    Welcome=Archori
```

```bash
[ K8s - Control cm ]: kubectl apply -f mysql-configmap.yaml 
configmap/mysql configured
[ K8s - Control cm ]: kubectl get cm
NAME               DATA   AGE
cmtest             1      92m
kube-root-ca.crt   1      193d
mysql              3      23m
mysql-config       2      76m
www-nginx          1      84m
[ K8s - Control cm ]: kubectl describe cm mysql
Name:         mysql
Namespace:    default
Labels:       app=mysql
Annotations:  <none>

Data
====
log:
----
1

lower:
----
1

my.cnf:
----
[mysqld]
Welcome=Archori



BinaryData
====

Events:  <none>
```

创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod-volume
spec:
  containers:
  - name: mysql
    image: busybox
    command: ["/bin/sh","-c","sleep 3600"]
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: mysql-config      # 连接到卷
      mountPath: /tmp/config  # 挂载到容器
  volumes:
  - name: mysql-config        # 卷名称
    configMap:                # 配置 ConfigMap 挂载属性
      name: mysql             # 连接 ConfigMap 并做成卷
  restartPolicy: Never
```

```bash
[ K8s - Control cm ]: kubectl apply -f mysql-pod-volume.yaml 
pod/mysql-pod-volume created
[ K8s - Control cm ]: kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
mysql-pod           1/1     Running   0          30m
mysql-pod-envfrom   1/1     Running   0          16m
mysql-pod-volume    1/1     Running   0          7s
```

登录容器查看挂载目录

```bash
[ K8s - Control cm ]: kubectl exec -it mysql-pod-volume -c mysql -- /bin/sh
/  cd /tmp/config
/tmp/config  ls
log     lower   my.cnf
/tmp/config  cat log
1
/tmp/config  cat lower
1
/tmp/config  cat my.cnf
[mysqld]
Welcome=Archori
```



### 热更新

```bash
kubectl edit configmap <cm_Name>
```

```bash
[ K8s - Control cm ]: kubectl edit cm mysql
configmap/mysql edited
```

登录节点检查更新情况

```bash
[ K8s - Control cm ]: kubectl exec -it mysql-pod-volume -c mysql -- /bin/sh
/  cd /tmp/config
/tmp/config  ls
log     lower   my.cnf
/tmp/config  cat ./*
2           # 检查到内容已被热更新
1
[mysqld]
Welcome=Archori
```



## Secret

可以简单理解为加密的 ConfigMap 资源，用于加密存储敏感数据。

### 关键字段

```yaml
apiVersion: v1
kind: Secret
metadata:
  name:
type:
data:
  <Key>: <Value>
  <Key>: <Value>
  ...
```

### 构建

#### 通过命令行

```bash
kubectl create secret generic <secret_Name> --from-literal=<Key>=<Value>
```

示例

```bash
[ K8s - Control secret ]: kubectl create secret generic mysql-password --from-literal=password=Archori**060626
secret/mysql-password created
[ K8s - Control secret ]: kubectl get secret
NAME             TYPE     DATA   AGE
mysql-password   Opaque   1      6s
[ K8s - Control secret ]: kubectl describe secret mysql-password
Name:         mysql-password
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  15 bytes
```

> [!WARNING]
>
> TYPE = Opaque 为 Base64 加密，可直接被解密。

创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  labels:
    app: myapp
spec: 
  containers:
  - name: myapp
    image: ikubenetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    env:
    - name: MYSQL_ROOT_PASSWORD  # 容器内变量名
      valueFrom:
        secretKeyRef:            # 使用加密的文件
          name: mysql-passwd     # secret 资源名
          key: password          # 使用的键名
```



#### 通过卷挂载

通过 Base64 加密明文

```bash
echo -n '<String>' | base64  # 加密
echo -n <String> | base64 -d # 解密
```

示例

```bash
[ K8s - Control secret ]: echo -n 'Archori' | base64
QXJjaG9yaQ==
[ K8s - Control secret ]: echo -n QXJjaG9yaQ== | base64 -d
Archori
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: QXJjaG9yaQ== # 写加密后的数据
  password: MDYwNjI2     # 写加密后的数据
```

```bash
[ K8s - Control secret ]: kubectl apply -f secret.yaml 
secret/mysecret created
[ K8s - Control secret ]: kubectl get secret
NAME             TYPE     DATA   AGE
mysecret         Opaque   2      10s
mysql-password   Opaque   1      41m
[ K8s - Control secret ]: kubectl describe secret mysecret
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
username:  7 bytes
```

创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-volume
spec:
  containers:
  - name: myapp
    image: registry.cn-beijing.aliyuncs.com/google_registry/myapp:v1
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - name: secret-volume    # 连接卷名
      mountPath: /etc/secret # 容器内挂载目录
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: mysecret   # 加密卷名
```

登录容器查看卷挂载以及变量解密

```bash
[ K8s - Control secret ]: kubectl apply -f pod-secret-volume.yaml 
pod/pod-secret-volume created
[ K8s - Control secret ]: kubectl get pods
NAME                READY   STATUS    RESTARTS   AGE
pod-secret-volume   1/1     Running   0          6s
[ K8s - Control secret ]: kubectl exec -it pod-secret-volume -- /bin/sh
cd /etc/secret
/etc/secret
cat ./*
060626
Archori
```



## RBAC

**基于角色的访问控制**



### 基本概念

#### 认证

Kubernetes 通过 APIserver 对外提供服务，为了安全，我们需要对访问 APIserver 的用户做认证。

#### 授权

认证通过后用户仅代表是一个被 APIserver 信任的用户，但用户具体在集群中的权限需要再进行授权操作。

#### 准入控制

当用户经过认证和授权，最后一步就是准入控制，Kubernetes 提供了多种准入控制，类似插件，为 APIserver 提供了良好的可扩展性，让它可以做变更和验证。

#### 基于角色授权

在 Kubernetes 中，角色拥有权限，通过让用户扮演角色，使用户获得权限，从而实现访问控制。



#### 用户

##### User

普通用户。

##### UserAccount

用户账户。是 Kubernetes 外部用户使用的账号，如 kubectl 访问集群须用到 UserAccount 用户，使用 kubeadm 安装的 Kubernetes，默认的 UserAccount 为 kubernetes-admin

##### ServiceAccount

服务账户。是 Pod 使用的账号，Pod 容器进程需要访问 APIserver 时用的就是 ServiceAccount 账户，ServiceAccount 仅局限在它所在 NameSpace，每个 NameSpace 创建时都会自动创建一个 default 服务账户，Pod 默认使用 default 这个账户。





#### 绑定

##### 基于 RoleBinding 到 Role

用户通过 RoleBinding 绑定到 Role，但只能在 RoleBinding 所在 NameSpace 中依据 Role 的描述行使权力。

特点： Role 和 RoleBinding 一一配对

##### 基于 RoleBinding 到 ClusterRole

用户通过 RoleBinding 绑定到 ClusteRole，但只能在 RoleBinding 所在 NameSpace 中依据 ClusteRole 的描述行使权力。

特点： ClusteRole 一对多于 RoleBinding

##### 基于 ClusterRoleBinding 到 ClusterRole

用户通过 ClusterRoleBinding 绑定到 ClusteRole，其中 ClusterRoleBinding 属于所有 NameSpace 。

特点： 没有名称空间限制



### 认证方式

#### 客户端认证

客户端认证也称为双向TLS认证， kubectl 在访问 apiserver 的时候，apiserver 也要认证 kubectl 是否合法，他们都通过ca根证书来进行验证。

#### Bearertoken

Bearertoken 可以理解为 apiserver 将一个密码通过了非对称加密的方式告诉了 kubectl ，然后通过该密码进行相互访问。

#### Serviceaccount

客户端证书认证和 Bearertoken 的两种认证方式，都是外部访问 apiserver 的时候使用的方式，那么我们这次说的 Serviceaccount 是内部访问 pod 和 apiserver 交互时候采用的一种方式。Serviceaccount 包括 namespace、token、ca，且通过目录挂载的方式给予 pod ，当 pod 运行起来的时候，就会读取到这些信息，从而使用该方式和 apiserver 进行通信。



### 创建

#### ServiceAccount



创建 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-test
  namespace: default
  labels:
    app: sa
spec: 
  serviceAccountName: sa-test  # 指定 ServiceAccount
  containers:
  - name: sa-tomcat
    ports:
    - containerPort: 80
    image: nginx
    imagePullPolicy: IfNotPresent
    command:
    - "sh"
    - "-c"
    - "sleep 3600"
```



#### Role

Role 是一组权限的集合，权限的描述**仅在自己所在名称空间**中行使。

关键字段

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name:            # Role 资源名称
  namespace:       # Role 所在名称空间
rules:
- apiGroups: [""]      # API 组列表
  resources: [""]      # 被授权资源种类
  resourceNames: []    # 被授权资源具体名称
    verbs: ["","",""]  # 授权项目明细
```

例

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-read
  namespace: rbac
rules:
- apiGroups: [""]
  resources: ["pods"]             # 种类为 Pods
  resourceNames: []               # 无指定，默认所有 Pods
    verbs: ["get","watch","list"] # 权限为查看
    
# 含义：定义一个名为 pod-read 的 Role 资源，描述对 rbac 名称空间下所有 Pod 具有查看权限
```



#### ClusterRole

ClusterRole 是一组权限的集合，权限的描述**在所有名称空间下**可用。

关键字段

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name:              # ClusterRole 资源名称
rules:
- apiGroups: [""]    # API 组列表
  resources: [""]    # 被授权资源种类
  resourceNames: []  # 被授权资源具体名称
  verbs: ["","",""]  # 授权项目明细
```

例

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secrets-clusterrole
rules:
- apiGroups: [""]
  resources: ["secrets"]        # 种类为 secrets
  resourceNames: []             # 空，默认所有 secrets
  verbs: ["get","watch","list"] # 权限为查看
 
# 含义：定义一个名为 secrets-clusterrole 的 ClusterRole 资源，描述对所有名称空间下 secrets 资源有访问权限
```



#### RoleBinding

关键字段

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name:       # RoleBinding 资源名称
  namespace:  # RoleBinding 名称空间
subjects:
- kind:       # 作用对象
    name:     # 对象名称
    apiGroup: # 对象 API 组
roleRef:
- kind:       # 绑定对象类型
    name:     # 绑定对象名称
    apiGroup: # 绑定对象 API 组
```

示例：User 通过 RoleBinding 到 Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-read-bind
  namespace: rbac
subjects:
- kind: User    # 作用于 User
    name: es    # User 名称为 es
    apiGroup: rbac.authorization.k8s.io
roleRef:
- kind: Role       # 绑定到 Role
    name: pod-read # Role 名称为 pod-read
    apiGroup: rbac.authorization.k8s.io
    
# 含义：创建一个名为 pod-read-bind 的 RoleBinding 资源，描述对名为 es 的用户拥有在 rbac 名称空间下且名为 pod-read 的 Role 资源所描述的所有权限（命名空间取交集）
```

示例：User 通过 RollBinding 到 ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: es-all-resource
  namespace: rbac
subjects:
- kind: User
    name: es
    apiGroup: rbac.authorization.k8s.io
roleRef:
- kind: ClusterRole
  name: cluster-admin
  
# 含义：创建一个名为 es-all-resource 的 RollBinding 资源，描述对名为 es 的用户拥有在 rbac 名称空间下且名为 cluster-admin 的 ClusterRole 资源所描述的所有权限（命名空间取交集）
```





#### ClusterRoleBinding

关键字段

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name:       # RoleBinding 资源名称
  namespace:  # RoleBinding 名称空间
subjects:
- kind:       # 作用对象
    name:     # 对象名称
    apiGroup: # 对象 API 组
roleRef:
- kind:       # 绑定对象类型
    name:     # 绑定对象名称
    apiGroup: # 绑定对象 API 组
```



#### 通过接口访问资源

多数资源可以用其名称的字符串表示，也就是 Endpoint 中的 URL 相对路径，例如 Pod 中日志是 `GET /api/v1/namespaces/{namespace}/pods/{podname}/log`

示例

创建角色

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: logs-reader
  namespace: test
rules:
- apiGroups: [""]
  resources: ["pods","pods/log"]
  verbs: ["get","list","watch"]
```

```bash
[ K8s - Control sa ]: kubectl apply -f role-test.yaml 
role.rbac.authorization.k8s.io/logs-reader created
[ K8s - Control sa ]: kubectl get role -n test
NAME          CREATED AT
logs-reader   2025-08-02T17:24:17Z
```

创建 ServiceAccount 服务账号

```bash
[ K8s - Control sa ]: kubectl create sa sa-test -n test
serviceaccount/sa-test created
[ K8s - Control sa ]: kubectl get sa sa-test -n test
NAME      SECRETS   AGE
sa-test   0         28s
```

创建 RoleBinding 绑定 sa-test 到 logs-reader

```bash
[ K8s - Control sa ]: kubectl create rolebinding sa-test-3 -n test --role=logs-reader --user=system:serviceaccount:default:sa-test
rolebinding.rbac.authorization.k8s.io/sa-test-3 created
[ K8s - Control sa ]: kubectl get rolebinding -n test
NAME        ROLE               AGE
sa-test-3   Role/logs-reader   16s
```

创建 Pod 使用 sa-test 这个 ServiceAccount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-test-pod
  namespace: test
  labels:
    app: sa
spec: 
  serviceAccountName: sa-test  # 使用 sa-test 用户
  containers:
  - name: sa-tomcat
    ports:
    - containerPort: 80
    image: nginx
    imagePullPolicy: IfNotPresent
```



### 控制

#### 命令行

使用命令行进行 RBAC 授权

```bash
# 使用帮助命令
kubectl create rolebinding --help

kubectl create rolebinding NAME --clusterrole=NAME|--role=NAME [--user=用户名] [--group=组名] [--serviceaccount=命名空间:服务账户名] [--dry-run=server|client|none]
```

#### ResourceQuota

**ResourceQuota** 即 **准入控制器** 是 Kubernetes 上内置的准入控制器，用来限制名称空间下资源的使用，防止 Pod 过多创建时导致的性能问题。

关键字段

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name:                     # ResourceQuota 资源名称
  namespace:                # ResourceQuota 名称空间
spec:
  hard:                     
    pods: ""                   # Pod 最大数量
    requests.cpu: ""           # request 总和
    requests.memory:           # request 总和
    limits.cpu: ""             # limit 总和
    limits.memory:             # limit 总和
    count/deployments.apps: "" # Deployment 最大数量
    persistentvolumeclaims: "" # PVC 最大数量
```

例子

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-test
  namespace: quota
spec:
  hard:
    pods: "6"
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 10Gi
    count/deployments.apps: "6"
    persistentvolumeclaims: "6"
```



#### limitRanger

**LimitRanger** 准入控制器用来控制命名空间下 Pod 在创建时默认的资源配额

关键字段

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-memory
  namespace: limit
spec:
  limits:
  - default:          # 在默认情况下
      cpu:
      memory:
    defaultRequest:   # 默认 request
      cpu:
      memory:
    min:              # 最小
      cpu:
      memory:
    max:              # 最大
      cpu:
      memory:
    maxLimitRequestRatio:  # 比率
      cpu:
      memory:
    type: Container        # 类型
```

例

```yaml
apiVersion: v1
kind: LimitRanger
metadata:
  name: cpu-memory
  namespace: limit
spec:
  limits:
  - default:
      cpu: 1000m
      memory: 1000Mi
    defaultRequest:
      cpu: 500m
      memory: 500Mi
    min:
      cpu: 500m
      memory: 500Mi
    max:
      cpu: 2000m
      memory: 2000Mi
    maxLimitRequestRatio:
      cpu: 4
      memory: 4
    type: Container
```



## Ingress



### 基本概念

**Ingress** 即 **七层负载均衡** 是 Kubernetes 中应用层资源，用于管理 ingress-controller 配置。

层级

```
UserAsk -> IngressController -> Services -> Pods
                  ^          
                  |
               Ingress
```

```bash
#1 部署 IngressController
#2 创建 Pod
#3 创建 Service 分组 Pod
#4 创建 Ingress 规则
```



### 关键字段

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: 
  namespace:
spec:
  ingressClassName:       # ClassName
  rules:                  # 规则
  - host:                 # 域名
    http:                 # 协议
      paths:
      - path: /
        pathType:
        backend:          # 后端设置
          service:
            name:         # 服务名称
            port:
              number:     # 使用端口号
```





### 高可用方案

IngressController 是集群流量的接入层，使用高可用方案非常重要。可以基于 Keepalive 实现 Nginx-ingress-controller 高可用。

IngressController 根据 Deployment + nodeSeletor + PodAntiAffinity 方式部署在 Kubernetes 指定的两个工作节点，通过对 nginx-ingress-controller Pod 共享宿主机 IP ，使用 Keepalive + Lvs 实现 nginx-ingress-controller 高可用。

```yaml

```



### HTTP 代理



创建四层代理，并部署后端 tomcat 服务

```yaml
apiVersion: v1
kind: Service
metadata:
  name: tomcat
  namespace: default
spec:
  selector:
    app: tomcat
    release: canary
  ports:
  - name: http
    targetPort: 8080
    port: 8080
  - name: ajp
    targetPort: 8009
    port: 8009
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deploy
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat
      release: canary
  template:
    metadata:
      labels:
        app: tomcat
        release: canary
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5.34-jre8-alpine 
        imagePullPolicy: IfNotPresent  
        ports:
        - name: http
          containerPort: 8080
          name: ajp
          containerPort: 8009
```

创建七层代理

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myapp
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - host: tomcat.lucky.com
    http:
      paths:
      - backend:
          service:
            name: tomcat
            port:
              number: 8080
        path: /
        pathType: Prefix
```

