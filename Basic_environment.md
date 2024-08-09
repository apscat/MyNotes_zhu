# 云平台部署(最简单)

### 本篇来源于 51CTO , 原文链接 : `https://blog.51cto.com/u_16213596/10305651` , 作者: mob64ca13fb6939

Centos7.5最小化安装后基础环境
```
配置 IP GTEWAY DNS ,上传软件包 Centos-7.repo 配置 yum 源 或 curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```
> 1. 安装管理软件 此处选择 宝塔
>> - `yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh`
>> - 保存登录地址 用户名 密码
> 2. 开放 8888 端口
>> - `firewall-cmd --zone=public --add-port=8888/tcp --permanent`
>> - 查看开放端口 `firewall-cmd --list-ports`



