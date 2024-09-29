# 单节点gpmall

## 本篇来源于 https://www.cnblogs.com/sh1ny2/p/14042842.html

```
上传 CentOS-7-x86_64-DVD-1511.iso , gpmall-cloud.tar.gz 
```

* ### 基础环境
- > 网络配置,`ip dns gateway netmask` 不修改,记得重启
> > 1. `vi /etc/sysconfig/network-scripts/ifcfg-eth1`
> > 2. 更改`BOOTPROTO=static`

- > 修改主机名为`mall`
> > 1. `hostnamectl set-hostname mall`
> > 2. `bash`

- > 配置主机名与ip映射,关闭防火墙和selinux,选择暂时关闭
> > 1. `vi /etc/hosts` 添加 `ip_you mall`

- > 挂载镜像,配置本地yum源,备份原文件
> > 1. `mount /root/CentOS-7-x86_64-DVD-1511.iso /opt/centos/`
> > 2. `vi /etc/yum.repos.d/local.repo`
```repo
[centos]
name=centos
baseurl=file:///opt/centos
gpgcheck=0
enabled=1
[gpmall-mall]
name=gpmall-mall
baseurl=file:///root/gpmall-repo
gpgcheck=0
enabled=1
```
> > 4. 清除缓存 查看yum列表

- > gpmall基础服务安装
> > 1. 安装java环境 `yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel`
> > 2. 安装redis缓存 `yum install redis -y` 若出现显示无包的情况可先安装 `EPEL` 仓库 `yum install epel-release -y`
> > 3. 安装Elasticsearch服务 `yum install elasticsearch -y` 也可不安装 若出现无包的情况可通过上传`elasticsearch-8.5.2-x86_64.rpm`进行安装
> > 4. 安装nginx服务 `yum install nginx -y`
> > 5. 安装MariaDB数据库 `yum install mariadb mariadb-server -y`
> > 6. 安装ZooKeeper服务
> > > -  `tar -xzvf gpmall-cloud/zookeeper-3.4.14.tar.gz -C /opt/`
> > > - `cd /opt/zookeeper-3.4.14/conf`
> > > - `mv zoo_sample.cfg zoo.cfg`
> > > - `cd ../bin/`
> > > - `sh zkServer.sh start`
> > 7. 安装Kafka服务
> > > - `cd /root/gpmall-cloud/`
> > > - `tar -xzvf kafka_2.11-1.1.1.tgz -C /opt/`
> > > - `cd /opt/kafka_2.11-1.1.1/bin`
> > > - `sh kafka-server-start.sh -daemon ../config/server.properties`
> > > - 通过 `jps` 或 `netstat -ntpl` 查看kafka是否启动
> > > > 安装net-tools工具包 `yum install net-tools -y`   `netstat -ntpl` 查看是否有9092端口 有则kafka成功启动
- > 配置
