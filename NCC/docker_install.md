# docker

* #### 基础准备

> `docker-repo` `Centos-7.repo` 上传
- 关闭防火墙 SElinux
- 清空 `/etc/yum.repos.d/` 下的所有文件,将`Centos-7.repo`移至该目录下,在该目录下编写 `local.repo` 文件,内容为:
```repo
[docker]
name=docker
baseurl=file:///root/docker-repo
gpgheck=0
enabled=1
```
