# Kubernetes

### 本篇来源于 作者: wenbin8 原文链接: `https://github.com/wenbin8/doc/blob/master/分布式/CloudNative/Kubernetes/01-Kubernetes集群搭建.md`

Centos7.5最小化安装后基础环境
```
建议三台centos , 2核2G起 , master worker1 worker2 , 过程中记得打快照
配置 IP GTEWAY DNS ,配置 yum 源 上传软件包 Centos-7.repo 替换 CentOS-Base.repo 或 curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

* ****

> 1. 更新并安装依赖(所有)
> > - `yum update -y`
> > - `yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp`

> 2. 安装 docker (所有)
> > 2.1 安装依赖
> > > `yum install yum-utils  device-mapper-persistent-data lvm2 -y`
> > > 
> > 2.2 设置 docker 仓库
> > > - `yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
> > > - `mkdir -p /etc/docker`
> > > - `vi /etc/docker/daemon.json `
```
{
  "registry-mirrors": ["IP"]
}
```
> > > - `systemctl daemon-reload`
> > > 
> > 2.3 安装 docker
> > > -  yum install docker-ce-18.09.0 docker-ce-cli-18.09.0 containerd.io -y
> > >   
> > 2.4 启动 docker
> > > - systemctl start docker && systemctl enable docker
> 3. 修改 hosts 文件
> > 3.1 master
> > > - `hostnamectl set-hostname master`
> > > - `vi /etc/hosts`
```yaml
192.168.100.10 master
192.168.100.11 worker1
192.168.100.12 worker2
```
> > 3.2 worker
> > > - `hostnamectl set-hostname worker1`
> > > - `hostnamectl set-hostname worker2`
> > > - `vi /etc/hosts`
```yaml
192.168.100.10 master
192.168.100.11 worker1
192.168.100.12 worker2
```
> > 3.3 三台机子 ping 通

> 4. 系统基础配置
> > 4.1 关闭防火墙 SELinux swap
> > > - `systemctl stop firewalld && systemctl disable firewalld`
> > > - `setenforce 0`
> > > - `sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config`
> > > - `swapoff -a`
> > > - `sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab`
> > 4.2 配置iptables的ACCEPT规则
> > >  - `iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT`
> > 4.3 设置系统参数
> > > - `vi /etc/sysctl.d/k8s.conf`
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```
> > > - `sysctl --system`

> 5. 安装 kubeadm  kubelet  kubectl
> > 5.1 配置 yum 源
> > > - `vi /etc/yum.repos.d/kubernetes.repo`
```yaml
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
> > > - `yum install -y kubeadm-1.14.0-0 kubelet-1.14.0-0 kubectl-1.14.0-0`
> > 5.2 docker
> > > - `vi /etc/docker/daemon.json`
```json
"exec-opts": ["native.cgroupdriver=systemd"],
```
> > > - `systemctl restart docker`
> > 5.3 kubectl 这里如果输出 directory not exist , 也说明是没问题的 继续即可
> > > - `sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
> > > - `systemctl enable kubelet && systemctl start kubelet`
> 6. proxy/pause/scheduler 等国内镜像
> > 6.1 查看 kubeadm 使用的镜像
> > > - `kubeadm config images list`
```
k8s.gcr.io/kube-apiserver:v1.14.0
k8s.gcr.io/kube-controller-manager:v1.14.0
k8s.gcr.io/kube-scheduler:v1.14.0
k8s.gcr.io/kube-proxy:v1.14.0
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.3.10
k8s.gcr.io/coredns:1.3.1
```
> > 6.2 解决国外镜像不能访问问题 , 创建kubeadm.sh脚本 , 用于拉取镜像/打tag/删除原有镜像
> > > - `vi kubeadm.sh`
```shell
#!/bin/bash

set -e

KUBE_VERSION=v1.14.0
KUBE_PAUSE_VERSION=3.1
ETCD_VERSION=3.3.10
CORE_DNS_VERSION=1.3.1

GCR_URL=k8s.gcr.io
ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/google_containers

images=(kube-proxy:${KUBE_VERSION}
kube-scheduler:${KUBE_VERSION}
kube-controller-manager:${KUBE_VERSION}
kube-apiserver:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
etcd:${ETCD_VERSION}
coredns:${CORE_DNS_VERSION})

for imageName in ${images[@]} ; do
  docker pull $ALIYUN_URL/$imageName
  docker tag  $ALIYUN_URL/$imageName $GCR_URL/$imageName
  docker rmi $ALIYUN_URL/$imageName
done
```
> > > - `sh kubeadm.sh`
> > > - `docker images`
> > > - `docker login --username=xxx registry.cn-hangzhou.aliyuncs.com`
> > > - 









