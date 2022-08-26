title: 记k8s在公网部署
date: 2020-10-29 10:26:59
tags: 
---

## 准备工作

### 1. 安装必备软件

```
yum install -y wget vim net-tools epel-release
```

### 2. 关闭swap

```
swapoff -a

# 永久禁用，打开/etc/fstab注释掉swap那一行。
sed -i 's/.*swap.*/#&/' /etc/fstab
```

### 3. 关闭selinux

```
# 临时禁用selinux
setenforce 0
# 永久关闭 修改/etc/sysconfig/selinux文件设置
sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
```

### 4. 关闭防火墙

```
systemctl disable firewalld
systemctl stop firewalld
```

## 安装Docker和配置代理

### 1. 配置yum源

```
## 配置默认源
## 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

## 下载阿里源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

## 刷新
yum makecache fast

## 配置k8s源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

## 重建yum缓存
yum clean all
yum makecache fast
yum -y update


```

### 2. 安装docker

```
yum -y install yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum -y install docker-ce
systemctl enable docker
systemctl start docker

```

### 3. 确保kubelet使用的cgroup driver 与 Docker的一致

```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

sudo systemctl daemon-reload sudo systemctl restart docker

```

### 4. 设置docker代理（核心步骤-如果有代理）

> 没有代理执行步骤5

```
mkdir /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://xxx"
Environment="HTTPS_PROXY=http://xxx"
Environment="NO_PROXY=localhost,127.0.0.1,localaddress,.localdomain.com"

systemctl daemon-reload && systemctl restart docker

```

### 5. 从国内仓库拉取镜像（核心步骤-如果没有代理）

```
## 查看集群初始化所需镜像及对应依赖版本号，列出的就是需要下载的镜像
kubeadm config images list

```

```
#!/bin/bash
images=(
kube-apiserver:v1.18.0
kube-controller-manager:v1.18.0
kube-scheduler:v1.18.0
kube-proxy:v1.18.0
pause:3.2
etcd:3.4.3-0
coredns:1.6.7
kubernetes-dashboard-amd64:v1.10.1
)
for imageName in ${images[@]};do
        docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
        docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
done
```

## 开始安装k8s



### 1. 安装kubeadm，kubelet等

```
yum -y install kubelet kubeadm kubectl kubernetes-cni
systemctl enable kubelet && systemctl start kubelet

```

### 2. 集群初始化

```
## master节点执行：
sudo kubeadm init \
 --apiserver-advertise-address 10.1.69.101 \
 --kubernetes-version=v1.15.0 \
 --pod-network-cidr=10.244.0.0/16

```

> 公网会卡在:

```
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
```

## 修改etcd.yaml

> 文件路径"/etc/kubernetes/manifests/etcd.yaml"

##### 修改前

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g90yzxelfhj30m00f8ac2.jpg)

##### 修改后

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g90z0cev0kj30m00f8dhs.jpg)

> 此处"xxx"为公网ip，要关注的是"--listen-client-urls"和"--listen-peer-urls"。需要把--listen-client-urls 和 --listen-peer-urls 都改成0.0.0.0：xxx

[来源](https://my.oschina.net/u/4389078/blog/3233116)

得到回复：

```

(...省略)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

## 保存好该命令，丢了不好找回。节点加入时需要
kubeadm join 10.1.69.101:6443 --token ou5pvo.qseafc4s8licblzy \
    --discovery-token-ca-cert-hash sha256:de9c10f11c50c074f212698b9d514fc12a9c1c4ffe70961aff89ac5e585f0663

```

### 3. 拷贝配置，给kubectl使用

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



### 4. 安装flannel网络

```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

## 查看flannal是否安装成功

sudo kubectl -n kube-system get po -l app=flannel -o wide

```

## 5. node节点加入集群

其他节点执行：

```
 kubeadm join 39.99.xxx.xxx:6443   --token w4tx2r.0dm194h24wlw8m8t --discovery-token-ca-cert-hash sha256:caae3b178b4172839269254f373b6eaa14514173b01e129e0b0fe7d64694dc39
```

## 清理安装

>  如果安装过程中出任何问题，可以重置后重新安装

```
sudo kubeadm reset
```

# Dashboard安装

```
# 安装dashboard，国内可以使用别的yaml源
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# 修改node为NodePort模式
kubectl patch svc -n kube-system kubernetes-dashboard -p '{"spec":{"type":"NodePort"}}'

# 查看服务(得知dashboard运行在443:32383/TCP端口)
kubectl get svc -n kube-system 
# --- 输出 ---
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   7h40m
kubernetes-dashboard   NodePort    10.111.77.210   <none>        443:32383/TCP            3h42m
# --- 输出 ---

# 查看dashboard运行在哪个node(得知dashboard运行在192.168.20.4)
kubectl get pods -A -o wide
# --- 输出 ---
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE     IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-fb8b8dccf-rn8kd                 1/1     Running   0          7h43m   10.244.0.2     master   <none>           <none>
kube-system   coredns-fb8b8dccf-slwr4                 1/1     Running   0          7h43m   10.244.0.3     master   <none>           <none>
kube-system   etcd-master                             1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-apiserver-master                   1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-controller-manager-master          1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kube-flannel-ds-amd64-l8c7c             1/1     Running   0          7h3m    192.168.20.5   master   <none>           <none>
kube-system   kube-flannel-ds-amd64-lcmxw             1/1     Running   1          6h50m   192.168.20.4   node1    <none>           <none>
kube-system   kube-flannel-ds-amd64-pqnln             1/1     Running   1          6h5m    192.168.20.3   node2    <none>           <none>
kube-system   kube-proxy-4kcqb                        1/1     Running   0          7h43m   192.168.20.5   master   <none>           <none>
kube-system   kube-proxy-jcqjd                        1/1     Running   0          6h5m    192.168.20.3   node2    <none>           <none>
kube-system   kube-proxy-vm9sj                        1/1     Running   0          6h50m   192.168.20.4   node1    <none>           <none>
kube-system   kube-scheduler-master                   1/1     Running   0          7h42m   192.168.20.5   master   <none>           <none>
kube-system   kubernetes-dashboard-5f7b999d65-2ltmv   1/1     Running   0          3h45m   10.244.1.2     node1    <none>           <none>
# --- 输出 ---
# 如果无法变成Running状态，可以使用以下命令排错
journalctl -f -u kubelet  # 只看当前的kubelet进程日志
### 提示拉取镜像失败，无法翻墙的可以使用以下方法预先拉取镜像
### 请在kubernetes-dashboard的节点上操作：
docker pull mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1
docker tag mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
docker rmi  mirrorgooglecontainers/kubernetes-dashboard-amd64:v1.10.1

```

获取token命令

```
## 获取登陆界面token
kubectl -n kube-system describe $(kubectl -n kube-system \
get secret -n kube-system -o name | grep namespace) | grep token

```

如果dashboard的token过期，以下脚本重新生成config文件

```
#!/bin/bash
TOKEN=$(kubectl -n kube-system describe secret default| awk '$1=="token:"{print $2}')
kubectl config set-credentials kubernetes-admin --token="${TOKEN}"

```

