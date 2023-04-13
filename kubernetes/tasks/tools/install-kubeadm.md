## Kubernetes

---

### 安装 `docker`

```bash
yum install -y docker
```

### 安装`kubectl`,`kubelet`,`kubeadm`

- `kubeadm`：用来初始化集群（`Cluster`）
- `kubelet`：运行在集群中的所有节点上，负责启动 `pod` 和 容器。
- `kubectl`：这个是 `Kubernetes` 命令行工具。通过 `kubectl` 可以部署和管理应用，查看各种资源，创建、删除和更新各种组件。

```bash
# 设置kubernetes源,默认源中没有kubectl,kubelet,kubeadm
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

```bash
# 开始安装
yum install -y kubelet kubeadm kubectl
# 设置开机自启并启动
systemctl enable kubelet && systemctl start kubelet
```



```bash
# !/bin/bash
for i in `kubeadm config images list`; do 
  imageName=${i#k8s.gcr.io/}
  docker pull registry.aliyuncs.com/google_containers/$imageName
  docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
  docker rmi registry.aliyuncs.com/google_containers/$imageName
done;
```



```bash
vi /etc/sysctl.conf
# 添加如下内容,保存后退出
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
# 生效配置
sysctl --system
```

```bash
# 关闭swap,不关闭无法启动
swapoff -a
# 修改配置,避免下次开机再次启用
vi /etc/fstab
# /dev/mapper/centos-swap swap swap default 0 0
```

```bash
# 初始化
kubeadm init --kubernetes-version=$(kubeadm version -o short)  --pod-network-cidr=10.244.0.0/16

# 初始化成功后输出如下信息
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.187.128:6443 --token 7b5kyj.m4ucnluzpbgl52ff \
    --discovery-token-ca-cert-hash sha256:0f889d168a5e86fc125234d23a8862396a628715357dce8bc2b0cb1f0c2c3496 
```

```bash
kubuctl applt -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

[ 从私有仓库拉取镜像 ](https://k8smeetup.github.io/docs/tasks/configure-pod-container/pull-image-private-registry/)

```bash
kubectl create secret docker-registry regsecret --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
```

