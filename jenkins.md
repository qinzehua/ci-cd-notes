# 安装 docker

```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y docker-ce

systemctl enable docker && systemctl start docker

mkdir -p /etc/docker
cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

# 安装 jenkins

```
ssh-keygen -t rsa -C "77593587@qq.com"

yum install java-1.8.0-openjdk* -y

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins -y

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json

systemctl enable jenkins
systemctl start jenkins

service firewalld start
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --permanent --add-port=50000/tcp
firewall-cmd --reload

curl 39.104.59.221:8080
```

## 实战 jenkins

### 增加 jenkins 访问 docker 的权限

```
gpasswd -a jenkins docker
newgrp docker

```

### 测试

![screenshot-39 104 170 121_8080-2021 08 11-21_40_52](https://user-images.githubusercontent.com/20763362/129058912-a1cb4b1f-4d33-4293-b10b-bde75a27e151.png)

### 安装 node 插件

![screenshot-39 104 170 121_8080-2021 08 11-23_38_00](https://user-images.githubusercontent.com/20763362/129060053-0c57b025-97cc-4d6c-a1e3-6bd4f5897533.png)

### 全局配置 node

![screenshot-39 104 170 121_8080-2021 08 11-23_54_59](https://user-images.githubusercontent.com/20763362/129062904-bbe53105-7032-49a4-9fdf-f201d9db7159.png)

### 项目配置 node（每个项目都需要单独添加 node 配置才能使用 node）

![screenshot-39 104 170 121_8080-2021 08 12-00_05_45 (1)](https://user-images.githubusercontent.com/20763362/129065308-b19dc46d-7c43-47b6-9396-08bb1ec8f081.png)

### 添加仓库地址，并第一次创建证书

![screenshot-39 104 170 121_8080-2021 08 12-00_25_45](https://user-images.githubusercontent.com/20763362/129066858-d885997a-a776-45d6-ab42-3df15eca7583.png)

### 创建证书

![screenshot-39 104 170 121_8080-2021 08 12-00_31_07](https://user-images.githubusercontent.com/20763362/129067685-73f6fcfb-b626-44b8-b5ac-a19a82fbf1d4.png)

### 更新 shell

![screenshot-39 104 170 121_8080-2021 08 12-19_57_29](https://user-images.githubusercontent.com/20763362/129193255-2be38f79-9faf-43ec-bb0b-a78ec273df7f.png)

# 部署 nexus 服务

```
cd /usr/local
wget https://dependency-fe.oss-cn-beijing.aliyuncs.com/nexus-3.29.0-02-unix.tar.gz
tar -zxvf ./nexus-3.29.0-02-unix.tar.gz
cd nexus-3.29.0-02/bin
./nexus start



```

## 访问 nexus 网页服务端口 8081，设置

![screenshot-39 104 170 121_8081-2021 08 13-00_47_18](https://user-images.githubusercontent.com/20763362/129236225-cef69499-3f9e-4b6e-9fc7-e3dc4b63d763.png)

## 选择创建仓库

![screenshot-39 104 170 121_8081-2021 08 12-23_50_41](https://user-images.githubusercontent.com/20763362/129228203-65e8362f-afac-42c5-a207-2f8b85bd422d.png)

## 选择 docker(hosted)

![screenshot-39 104 170 121_8081-2021 08 12-23_54_01](https://user-images.githubusercontent.com/20763362/129228484-bd978893-1d70-48c3-9909-3bdf5da614a3.png)

## 仓库基本设置，然后点击创建

![screenshot-39 104 170 121_8081-2021 08 13-00_24_04](https://user-images.githubusercontent.com/20763362/129232893-8d85f4c4-f75c-43e1-8688-cd7cd4cd79e7.png)

## 访问刚刚创建的仓库页面

![screenshot-39 104 170 121_8081-2021 08 13-00_41_33](https://user-images.githubusercontent.com/20763362/129235450-70fcf475-96b8-4853-8652-320b95ba161e.png)

## 配置 docker 让 http 协议也可以推送镜像

```
cat <<EOF > /etc/docker/daemon.json
{
  "insecure-registries": ["39.104.170.121:8082"],
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF

systemctl daemon-reload
systemctl restart docker

```

## 配置推送镜像时，需要登录 docker 的账号密码

![screenshot-39 104 170 121_8080-2021 08 14-11_09_35](https://user-images.githubusercontent.com/20763362/129432545-3edccf8a-1f63-45fe-bbcd-c816295f9ff8.png)
![screenshot-39 104 170 121_8080-2021 08 14-11_14_37](https://user-images.githubusercontent.com/20763362/129432576-bf45b7a2-9bcc-4df7-9d8f-e8d28c1cd488.png)
![screenshot-39 104 170 121_8080-2021 08 14-11_16_32](https://user-images.githubusercontent.com/20763362/129432608-4c0bbddb-c337-4c58-bd5e-6406131ec497.png)

## 更新 shell 任务

```
npm config set registry https://registry.npm.taobao.org
npm install
npm run build
time=$(date "+%Y%m%d%H%M%S")
docker build -t 39.104.170.121:8082/reactproject:$time .
docker login -u $DOCKER_LOGIN_USERNAME -p $DOCKER_LOGIN_PASSWORD 39.104.170.121:8082
docker push 39.104.170.121:8082/reactproject:$time
```

![screenshot-39 104 170 121_8080-2021 08 14-11_32_09](https://user-images.githubusercontent.com/20763362/129432864-ff3d99bb-1437-45f8-9634-2a9d0b86b910.png)
