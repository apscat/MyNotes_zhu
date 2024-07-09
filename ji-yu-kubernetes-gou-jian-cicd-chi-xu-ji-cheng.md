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
