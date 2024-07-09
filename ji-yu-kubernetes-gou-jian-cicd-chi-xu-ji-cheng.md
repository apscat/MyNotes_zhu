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
> > >  - docker login -uadmin -pHarbor12345 IP 
> > >  - docker load -i BlueOcean/images/maven_latest.tar  
> > >  - docker tag maven IP/library/maven 
> > >  - docker push IP/library/maven
> > >  - docker load -i BlueOcean/images/java_8-jre.tar  
> > >  - docker load -i BlueOcean/images/jenkins_jenkins_latest.tar   
> > >  - docker load -i BlueOcean/images/gitlab_gitlab-ce_latest.tar

* #### 部署 Jenkins

> 1. 安装 Jenkins  
>
> > 1. 新建命名空间 `kubectl create ns devops`
> > ```
> > 部署Jenkins需要使用到一个拥有相关权限的serviceAccount，名称为jenkins-admin，可以给jenkins-admin赋予一些必要的权限，也可以直接绑定一个cluster-admin的集群角色权限，此处选择给予集群角色权限。
> > ```
> > 2. 编写资源清单文件 `vi jenkins-deploy.yaml `
```yaml 
apiVersion: v1   
kind: Service  
metadata: 
  name: jenkins
  labels:
    app: jenkins
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080
    targetPort: 8080
    nodePort: 30880
  - name: agent
    port: 50000
    targetPort: agent
    nodePort: 30850
  selector:
    app: jenkins

---
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: jenkins
  labels:
    app: jenkins
spec:
  selector:
    matchLabels: 
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins-admin
      containers:
      - name: jenkins
        image: jenkins/jenkins:latest 
        imagePullPolicy: IfNotPresent
        securityContext: 
          runAsUser: 0
          privileged: true
        ports:
        - name: http
          containerPort: 8080
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkinshome
        - mountPath: /usr/bin/docker
          name: docker
        - mountPath: /var/run/docker.sock
          name: dockersock
        - mountPath: /usr/bin/kubectl
          name: kubectl
        - mountPath: /root/.kube
          name: kubeconfig
      volumes:
      - name: jenkinshome
        hostPath:
          path: /home/jenkins_home
      - name: docker
        hostPath:
          path: /usr/bin/docker
      - name: dockersock
        hostPath:
          path: /var/run/docker.sock
      - name: kubectl
        hostPath: 
          path: /usr/bin/kubectl
      - name: kubeconfig
        hostPath:
          path: /root/.kube 
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkinshome 
  annotations:
    volume.beta.kubernetes.io/storage-class: local-path
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1024Mi 
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin 
  labels:
    name: jenkins
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata: 
  name: jenkins-admin
  labels:
    name: jenkins
subjects: 
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: default
roleRef:
  kind: ClusterRole
  name: cluster-admin 
  apiGroup: rbac.authorization.k8s.io 
```

> > 3. `kubectl -n devops apply -f jenkins-deploy.yaml` 
> > 4. `kubectl -n devops get pods`
> > 5. `kubectl -n devops cp BlueOcean/plugins/ jenkins-cc97fd4fc-v5dh2:/var/jenkins_home`
> > 6. `kubectl -n devops rollout restart deployment jenkins`
> 2. 访问 Jenkins
> > 1.  查看 Jenkins Service 端口 `kubectl -n devops get svc`
> > 2.  Web 访问 http://master_IP:30880
> > 3.  获取 Jenkins 密码 `kubectl -n devops exec deploy/jenkins -- cat /var/jenkins_home/secrets/initialAdminPassword`
> > 4.  输入密码后单击“继续”按钮,选择“跳过插件安装”,如图：
> > > ![插件安装](https://bbs-img.huaweicloud.com/blogs/img/20231024/1698142010333732951.png)
> > 6.  安装完成后进入用户创建页面，创建一个Username 为 jenkins，密码为 000000 ，其他项随便填 , 如图：
> > > ![创建账户](https://bbs-img.huaweicloud.com/blogs/img/20231025/1698229461407799933.JPG)
> > > ![创建账户](https://bbs-img.huaweicloud.com/blogs/img/20231024/1698142491379952155.JPG)
> > > 
> > 7.单击“开始使用 Jenkins ”按钮并使用新创建的用户登录 Jenkins
* #### 部署 Gitlab##
> ```
> GitLab是利用Ruby on Rails一个开源的版本管理系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。与GitHub类似，GitLab能够浏览源代码，管理缺陷和注释，可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库，团队成员可以利用内置的简单聊天程序（Wall）进行交流。GitLab还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。本项目GitLab与Harbor共用一台服务器。
> ```
> 1. 编写 Gitlab 清单文件 `vi gitlab-deploy.yaml `
> 2. 
