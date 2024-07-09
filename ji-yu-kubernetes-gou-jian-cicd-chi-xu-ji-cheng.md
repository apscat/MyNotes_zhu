# 基于 Kubernetes 构建 CI/CD 持续集成

```
部署Kubernetes,上传软件包 `BlueOcean.tar.gz`
```

* #### 部署 Harbor

> 1. 解压 BlueOcean.tar.gz , 安装docker-compose 
>
> > 1. `tar -zxvf BlueOcean.tar.gz`
> > 2. `cp BlueOcean/tools/docker-compose-Linux-x86_64 /usr/bin/docker-compose` 
> > 3. `docker-compose version` 

> 2. 安装 Harbor , 默认账号密码 `admin` `Harbor12345` , `http://master_ip` 
>
> > 1. tar -zxf BlueOcean/harbor-offline-installer.tar.gz -C /opt/
> > 2. sh /opt/harbor/install.sh
> > 3. 登录后新建 `springcloud` , 访问级别设置为公开
> > 4. 上传镜像到 Harbor 
> > > docker login -uadmin -pHarbor12345 IP
> > > docker load -i BlueOcean/images/maven_latest.tar 
> > > docker tag maven IP/library/maven
> > > docker push IP/library/maven
> > > docker load -i BlueOcean/images/java_8-jre.tar
> > > docker load -i BlueOcean/images/jenkins_jenkins_latest.tar
> > > docker load -i BlueOcean/images/gitlab_gitlab-ce_latest.tar

