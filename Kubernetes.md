# Kubernetes

### 本篇来源于 作者: wenbin8 原文链接: `https://github.com/wenbin8/doc/blob/master/%E5%88%86%E5%B8%83%E5%BC%8F/CloudNative/Kubernetes/01-Kubernetes%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA.md`

Centos7.5最小化安装后基础环境
```
建议三台centos , 2核2G起 , master worker1 worker2
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

> 
