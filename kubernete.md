# 1 安装必要插件, 以及相关必要设置

```
yum install nptdate -y
systemctl stop firewalld & systemctl disable firewalld
setenforce 0
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab

cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system


ln -snf /usr/share/zoneinfo/Shanghai /etc/localtime

bash -c "echo 'Asia/Shanghai' > /etc/timezone"

ntpdate ntp1.aliyun.com

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


cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
      https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


yum install -y kubelet-1.20.0 kubeadm-1.20.0 kubectl-1.20.0

systemctl enable kubelet && systemctl start kubelet

echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

echo '172.19.223.176  k8smaster' >> /etc/hosts
echo '172.19.223.171  k8snode' >> /etc/hosts

```

###

拷贝 linux 目录下的配置文件到 k8smaster 服务器

# 2 安装 k8s 相关 images, 安装 Flannel (这两个操作在 master)

```
kubeadm config images list --config init-kubeadm.conf

kubeadm config images pull --config init-kubeadm.conf

kubeadm init --config init-kubeadm.conf

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config


wget https://img.zhufengpeixun.com/kube-flannel.yml
docker pull quay.io/coreos/flannel:v0.13.0-rc2
kubectl apply -f kube-flannel.yml

```

```
scp $HOME/.kube/config root@172.19.223.171:~/
```

# 3 Node 节点操作

```
mkdir -p $HOME/.kube
sudo mv $HOME/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 在 master 节点获取 node 加入的 token

```
kubeadm token create --print-join-command
```

# 4 存储机密信息

## 4.1 Opaque 类型

### 4.1.1 通过命令

```
kubectl create secret generic my-secret-opaque --from-literal=username=admin --from-literal=password=123

kubectl get secret
```

### 4.1.2 通过声明式

```
kubectl apply -f ./secrets/secret-opaque-file.yaml
```

### 4.1.3 查看(kubectl get secret secret 名字 -o json)

```
kubectl get secret my-secret-opaque  -o json
kubectl get secret my-secret-opaque-file  -o json
```

## 4.2 私有镜像仓库认证

### 4.2.1 命令行创建(docker-registry 表示类型, my-private-auth 表示 secret 名字)

```
kubectl create secret docker-registry my-registry-auth --docker-username=admin --docker-password=123 --docker-email=admin@example.org --docker-server=39.104.170.121:8082
```

### 4.2.2 声明式

```
kubectl apply -f ./secrets/secret-registry-file.yaml
```

## 4.3 使用

### 4.3.1 volume 挂载

修改 deployment-user-v1.yaml

```diff
   selector:
     matchLabels:
       app: pod-user-v1
-  replicas: 3 #声明pod的副本数量
+  replicas: 1 #声明pod的副本数量
   template:
     metadata:
       labels:
         app: pod-user-v1
     spec: #描述的是pod内的容器信息
+      volumes:
+        - name: my-secret-opaque-volume # 数据卷名称
+          secret:
+            secretName: my-secret-opaque # secret 名字
       containers:
         - name: nginx
           image: registry.cn-beijing.aliyuncs.com/zhangrenyang/nginx:user-v1
+          volumeMounts:
+            - name: my-secret-opaque-volume
+              mountPath: /secret-opaque
+              readOnly: true
           ports:
             - containerPort: 80 #容器内映射的端口号
```

### 4.3.2 环境变量注入

修改 deployment-user-v2.yaml

```diff
    spec: #描述的是pod内的容器信息
       containers:
         - name: nginx
+          env:
+            - name: USERNAME
+              valueFrom:
+                secretKeyRef:
+                  name: my-secret-opaque-file
+                  key: username
+            - name: PASSWORD
+              valueFrom:
+                secretKeyRef:
+                  name: my-secret-opaque-file
+                  key: password
           image: registry.cn-beijing.aliyuncs.com/zhangrenyang/nginx:user-v2
           ports:
             - containerPort: 80 #容器内映射的端口号
```

### 4.3.3 Docker 私有库认证

deployment-user-v3.yaml

```diff
       labels:
         app: pod-user-v3
     spec:
+      imagePullSecrets:
+        - name: my-registry-auth-file
       containers:
-        - name: nginx
-          image: registry.cn-beijing.aliyuncs.com/zhangrenyang/nginx:user-v2
+        - name: reactproject
+          image: 39.104.170.121:8082/reactproject:20210814113053
           ports:
             - containerPort: 80 #容器内映射的端口号
```

```
kubectl apply -f /root/deploy/user/deployment-user-v1.yaml
kubectl apply -f /root/deploy/user/deployment-user-v2.yaml
kubectl apply -f /root/deploy/user/deployment-user-v3.yaml
kubectl apply -f /root/deploy/pay/deployment-pay-v1.yaml

kubectl apply -f /root/deploy/user/service-user-v1.yaml
kubectl apply -f /root/deploy/user/service-user-v2.yaml
kubectl apply -f /root/deploy/user/service-user-v3.yaml
kubectl apply -f /root/deploy/pay/service-pay-v1.yaml
```

安装 ingress,启动 ingress 的不同配置

```
kubectl apply -f /root/deploy.yaml
kubectl apply -f /root/ingress.yaml
kubectl apply -f /root/ingress-gray.yaml

```

```
阿里云需要开发过的端口号(默认)， (根据创建服务的具体设置开放)
jinkins: 8080
ingress: 31234, 31235
nexus 服务端口: 8081
nexus docker 仓库端口: 8082
```

# 6 统一管理服务环境变量

## 6.1 创建

### 6.1.1 命令行创建

```
kubectl create configmap mysql-config --from-literal=MYSQL_HOST=127.0.0.1 --from-literal=MYSQL_PORT=3306
```

### 查看

```
kubectl get configmap mysql-config  -o json
```

### 6.1.2 配置清单创建

```
kubectl apply -f ./configmaps/mysql-config-file.yaml
```

### 6.1.3 文件创建

```
kubectl create configmap env-from-file --from-file=./configmaps/env.config
```

### 6.1.4 目录创建

```
kubectl create configmap env-from-dir --from-file=./configmaps/config
```
