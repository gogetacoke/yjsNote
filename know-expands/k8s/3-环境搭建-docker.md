# 主机准备

| 主机名 | IP            | 配置 |
| ------ | ------------- | ---- |
| master | 192.168.10.10 | 2c3g |
| node1  | 192.168.10.11 | 2c3g |
| node2  | 192.168.10.12 | 2c3g |
| node3  | 192.168.10.13 | 2c3g |
| node4  | 192.168.10.14 | 2c3g |

#  初始环境准备

> 几台主机均能访问公网

```shell
# 每台主机按照上方设置主机名字
]#hostnamectl set-hostname 主机名
# 将此脚本复制到每台机器上执行
]#vim init.sh
#!/bin/bash
# 安装docker
dnf -y install docker-ce
systemctl enable --now docker


# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭selinux
setenforce 0  # 临时
sed -i 's/enforcing/disabled/' /etc/selinux/config  # 永久


# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久

# 关闭完swap后，一定要重启一下虚拟机！！！


# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.10.10 master
192.168.10.11 node1
192.168.10.12 node2
192.168.10.13 node3
192.168.10.14 node4
192.168.10.30 harbor
EOF


# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system  # 生效


# 时间同步；无法访问公网就本地搭建时间服务器，这里使用的网络时间
dnf -y install chrony
vim /etc/chrony.conf
3 server ntp.aliyun.com iburst
systemctl enable start chronyd



# 配置阿里云kubernetes仓库
cat >> /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


# 安装控制组件
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6

systemctl enable kubelet

# 配置关闭 Docker 的 cgroups，修改 /etc/docker/daemon.json，加入以下内容
mkdir -p /etc/docker
vim /etc/docker/daemon.json
{
    "registry-mirrors": [ "https://harbor:443", "http://registry:5000"],
    "insecure-registries":["harbor:443", "registry:5000"],
    "exec-opts": ["native.cgroupdriver=systemd"]
}

# 重启 docker
systemctl daemon-reload
systemctl restart docker
```

# 部署Master

> 再master节点执行
>
> 这里初始化需要去 registry.aliyuncs.com 拉去镜像，需要保证又网络的情况下执行，不然都自行将镜像下载下来上传到harbor仓库，这里修改为harbor仓库地址即可

```sh
# 查询kubelet状态，运行后才执行下方操作
]#systemctl status kubelet

# 在 Master 节点下执行
kubeadm init \
      --apiserver-advertise-address=192.168.10.10\
      --image-repository registry.aliyuncs.com/google_containers \
      --kubernetes-version v1.23.6 \
      --service-cidr=10.96.0.0/12 \
      --pod-network-cidr=10.244.0.0/16
      
          
# 上方执行完后出下下方这一串，表示初始化成功
# 安装成功后，复制如下配置并执行
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config



kubectl get nodes
```

# 生成节点加入密钥

```sh

kubeadm token create --ttl=0 --print-join-command
# 生成如下命令，复制到各个node节点上执行
kubeadm join 192.168.10.10:6443 --token yyxvjm.hkwq0wm7webykxiu --discovery-token-ca-cert-hash sha256:ab21191aa16b754bcb1c2a5b8cc062223e8da999d4352b12997114c4846e1d74 

# 查询pod;能查询到节点就行，现在处于NoReady状态
kubectl get nodes
```

# 部署CNI插件

```sh
# master主机上执行；每没有网络就去网上下载
curl https://calico-v3-25.netlify.app/archive/v3.25/manifests/calico.yaml -O

# 查询需要的镜像
grep image calico.yaml 

# 将docker.io替换掉
sed -i 's#docker.io/##g' calico.yaml

# 开始安装;执行后需要漫长的等待，网络好还行
kubectl apply -y calico.yaml
# 查询安装情况;所有都为running才算成功
kubectl get pods -n kube-system
# 某个pod一直处于pending或其余异常，查看日志
kubectl describe pods -n kube-system 处于pending的pod NAME
```

# 测试

```sh
# 全部节点处于ready，才算部署成功
]# kubectl get nodes
NAME     STATUS   ROLES                  AGE     VERSION
master   Ready    control-plane,master   4h11m   v1.23.6
node1    Ready    <none>                 3h48m   v1.23.6
node2    Ready    <none>                 3h45m   v1.23.6
node3    Ready    <none>                 3h44m   v1.23.6
node4    Ready    <none>                 3h44m   v1.23.6

# 拉去harbor仓库中的镜像测试；在harbor仓库创建一个公开仓库myos，上传一个httpd镜像
]#kubectl run web --image=myos:httpd
]# kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
web    1/1     Running   0          88m
```



