## 安装 docker

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

## 安装 jenkins

```
ssh-keygen -t rsa -C "77593587@qq.com"

yum install java-1.8.0-openjdk* -y

sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

yum install jenkins -y

sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/lib/jenkins/updates/default.json && sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /var/lib/jenkins/updates/default.json

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
