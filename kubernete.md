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

echo '172.19.223.172  k8smaster' >> /etc/hosts
echo '172.19.223.171  k8snode' >> /etc/hosts

```

### 3. Master

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

#### 3.5 安装 Flannel (这个操作在 master)

#### 3.7 node 节点配置

### master

`scp $HOME/.kube/config root@172.19.223.171:~/`

```
mkdir -p $HOME/.kube
sudo mv $HOME/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

`kubeadm token create --print-join-command`

```
kubectl apply -f /root/deploy/user/deployment-user-v1.yaml
kubectl apply -f /root/deploy/user/deployment-user-v2.yaml
kubectl apply -f /root/deploy/user/service-user-v1.yaml
kubectl apply -f /root/deploy/user/service-user-v2.yaml
kubectl apply -f /root/deploy/pay/deployment-pay-v1.yaml
kubectl apply -f /root/deploy/pay/service-pay-v1.yaml

kubectl apply -f /root/deploy.yaml
kubectl apply -f /root/ingress.yaml
kubectl apply -f /root/ingress-gray.yaml
```
