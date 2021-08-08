#### 2.1 安装必要插件

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
```

#### 2.5 统一系统时间

`ln -snf /usr/share/zoneinfo/Shanghai /etc/localtime`

`bash -c "echo 'Asia/Shanghai' > /etc/timezone"`

`ntpdate ntp1.aliyun.com`

#### 2.6 安装 docker

https://www.cnblogs.com/lifan1998/p/14326769.html

```
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y docker-ce

systemctl enable docker && systemctl start docker
```

```
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

#### 2.7 安装 k8s 组件

#### 2.7.1 切换阿里云的源

```
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
```

#### 2.7.2 安装 kubenenter 组件

`yum install -y kubelet-1.20.0 kubeadm-1.20.0 kubectl-1.20.0`

`systemctl enable kubelet && systemctl start kubelet`

#### 2.7.3 设置 bridege-nf-call-iptables

`echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables`

### 3. Master

#### 3.1 修改主机名为 k8smaster 节点名为 k8snode

主机

`hostnamectl set-hostname k8smaster`

节点

`hostnamectl set-hostname k8snode`

#### 3.2 配置 kubernetes 初始化文件 (这个操作仅在 master)

`kubeadm config print init-defaults > init-kubeadm.conf`

`vi ./init-kubeadm.conf`

修改:

```
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers

advertiseAddress: master 地址
```

增加

```
networking:

podSubnet: 10.244.0.0/16
```

修改

```
name: k8smaster
```

#### 3.3 拉取其他组件 (这个操作仅在 master)

```
//查看缺少的组件
kubeadm config images list --config init-kubeadm.conf

// 拉取缺少的组件
kubeadm config images pull --config init-kubeadm.conf

https://blog.51cto.com/8999a/2784605
```

#### 3.4 初始化 kubernetes (这个操作仅在 master)

```
kubeadm init --config init-kubeadm.conf
```

得到结果

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.19.223.172:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:a2b88d4aafaf22dbedf69d34aa142c518d3b450232c5a2b4a886476aba771704
```

#### 3.5 安装 Flannel (这个操作在 master)

第一种

```
vi kube-flannel.yml
// 内容为此链接的附录 https://segmentfault.com/a/1190000037682150

docker pull quay.io/coreos/flannel:v0.13.0-rc2
kubectl apply -f kube-flannel.yml
```

第二种

```
wget https://img.zhufengpeixun.com/kube-flannel.yml
docker pull quay.io/coreos/flannel:v0.13.0-rc2
kubectl apply -f kube-flannel.yml
```

#### 3.7 node 节点配置

`hostnamectl set-hostname k8snode`

#### 3.8 拷贝 Master 节点配置文件 (然后执行 3.5 的操作)

- 将 master 节点的配置文件拷贝到 k8snode 节点

  `scp $HOME/.kube/config root@172.19.223.171:~/`

- 在 k8snode 节点归档配置文件

```
mkdir -p $HOME/.kube
sudo mv $HOME/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

创建服务（例子）

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pod,svc

```

创建 deployment

```
mkdir deploy && cd ./deploy
vi deployment-user-v1.yaml
kubectl apply -f deployment-user-v1.yaml

```

创建 service

```
vi service-user-v1.yaml
kubectl apply -f service-user-v1.yaml
```

安装 ingress

```
wget https://img.zhufengpeixun.com/deploy.yaml

vi ./deploy.yaml
增加:
spec:
  ports:
    - port: 80
      nodePort: 31234
    - port: 443
      nodePort: 31235

替换:
image: registry.cn-hangzhou.aliyuncs.com/bin_x/nginx-ingress:v0.34.1@sha256:80359bdf124d49264fabf136d2aecadac729b54f16618162194356d3c78ce2fe

kubectl apply -f deploy.yaml

```

启动 ingress

```
vi ingress.yaml

```
