---
title: 2005231550@使用kubeadm在Centos8上部署kubernetes1.18
date: 2020-05-23 15:49:18
tags: K8S
---
# 1 系统准备

查看系统版本

```shell
[root@master01 ~]# cat /etc/centos-release
CentOS Linux release 8.1.1911 (Core)
```

配置网络

```shell
[root@master01 network-scripts]# cd /etc/sysconfig/network-scripts/
[root@master01 network-scripts]# ls
ifcfg-eth0  ifcfg-Wired_connection_1  route-eth0
[root@master01 network-scripts]# vim ifcfg-eth0 

BOOTPROTO=static
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME="System eth0"
UUID=5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03
IPADDR=192.168.1.201
PREFIX=24
GATEWAY=192.168.1.200
DNS1=114.114.114.114
PEERDNS=no

```

添加阿里源

```shell
[root@localhost ~]# rm -rfv /etc/yum.repos.d/*
[root@localhost ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

配置主机名

```shell
[root@master01 network-scripts]# hostnamectl set-hostname master01.cluee.tech^C
[root@master01 network-scripts]# hostnamectl
   Static hostname: master01.cluee.tech
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 6a58370cff464e84bf828dff79fa85c9
           Boot ID: 8424b77040f544bdac79a50f3507e90e
    Virtualization: microsoft
  Operating System: CentOS Linux 8 (Core)
       CPE OS Name: cpe:/o:centos:centos:8
            Kernel: Linux 4.18.0-147.el8.x86_64
      Architecture: x86-64
[root@master01 network-scripts]# 
```
临时关闭swap分区, 重启失效

```shell
swapoff -a
```

永久关闭swap分区

```shell
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

配置内核参数，将桥接的IPv4流量传递到iptables的链

```shell
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

# 2 安装常用包
```shell
[root@master01 ~]# yum install vim bash-completion net-tools gcc -y
```

# 3 使用aliyun源安装docker-ce
```shell
[root@master01 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2
[root@master01 ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@master01 ~]# yum -y install docker-ce
```
安装docker-ce如果出现以下错
```shell
[root@master01 ~]# yum -y install docker-ce
CentOS-8 - Base - mirrors.aliyun.com                                                                               14 kB/s | 3.8 kB     00:00
CentOS-8 - Extras - mirrors.aliyun.com                                                                            6.4 kB/s | 1.5 kB     00:00
CentOS-8 - AppStream - mirrors.aliyun.com                                                                          16 kB/s | 4.3 kB     00:00
Docker CE Stable - x86_64                                                                                          40 kB/s |  22 kB     00:00
Error:
 Problem: package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is excluded
  - package containerd.io-1.2.13-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.el7.x86_64 is excluded
  - package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```
解决方法
```shell
[root@master01 ~]# wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
[root@master01 ~]# yum install containerd.io-1.2.6-3.3.el7.x86_64.rpm
```
然后再安装docker-ce即可成功

添加aliyundocker仓库加速器
```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://x211rrqs.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 4 安装kubectl、kubelet、kubeadm

添加阿里kubernetes源

```shell
[root@master01 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装

```shell
[root@master01 ~]# yum install kubectl kubelet kubeadm
[root@master01 ~]# systemctl enable kubelet
```

# 5 初始化k8s集群

# kubeadm init

```shell
kubeadm init --kubernetes-version=v1.18.2 \
--image-repository registry.aliyuncs.com/google_containers \
--apiserver-advertise-address=192.168.1.201 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--ignore-preflight-errors=Swap
```

api server地址就是master本机IP、pod的网段为: 10.244.0.0/16

这一步很关键，由于kubeadm 默认从官网k8s.grc.io下载所需镜像，国内无法访问，因此需要通过–image-repository指定阿里云镜像仓库地址。

```shell
W0523 16:45:34.272700    3471 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.2
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[WARNING Hostname]: hostname "master201.cluee.tech" could not be reached
	[WARNING Hostname]: hostname "master201.cluee.tech": lookup master201.cluee.tech on 114.114.114.114:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [master201.cluee.tech kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.201]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [master201.cluee.tech localhost] and IPs [192.168.1.201 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [master201.cluee.tech localhost] and IPs [192.168.1.201 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0523 16:45:39.616901    3471 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0523 16:45:39.618174    3471 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 22.001953 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master201.cluee.tech as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node master201.cluee.tech as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 3yhf1l.1cn4avvvx9sqrfk2
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.201:6443 --token 3yhf1l.1cn4avvvx9sqrfk2 \
    --discovery-token-ca-cert-hash sha256:b46729b6a0d7be0c956953e06de0f0bb8c7040659d35cbe11b4c2cc83b931bcd

```

记录生成的最后部分内容，此内容需要在其它节点加入Kubernetes集群时执行。
根据提示创建kubectl

```shell
[root@master01 ~]#  mkdir -p $HOME/.kube
[root@master01 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@master01 ~]#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

执行下面命令，使kubectl可以自动补充

```shell
source <(kubectl completion bash)
```

查看节点，pod
```shell
[root@master01 ~]# kubectl get node
NAME                STATUS     ROLES    AGE     VERSION
master01.paas.com   NotReady   master   2m29s   v1.18.0
[root@master01 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-7ff77c879f-fsj9l                    0/1     Pending   0          2m12s
kube-system   coredns-7ff77c879f-q5ll2                    0/1     Pending   0          2m12s
kube-system   etcd-master01.paas.com                      1/1     Running   0          2m22s
kube-system   kube-apiserver-master01.paas.com            1/1     Running   0          2m22s
kube-system   kube-controller-manager-master01.paas.com   1/1     Running   0          2m22s
kube-system   kube-proxy-th472                            1/1     Running   0          2m12s
kube-system   kube-scheduler-master01.paas.com            1/1     Running   0          2m22s
[root@master01 ~]#
```

node节点为NotReady，因为corednspod没有启动，缺少网络pod

# 6 安装flannel网络

For Kubernetes v1.7+ 

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

查看pod和node

```shell
[root@master01 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-555fc8cc5c-k8rbk    1/1     Running   0          36s
kube-system   calico-node-5km27                           1/1     Running   0          36s
kube-system   coredns-7ff77c879f-fsj9l                    1/1     Running   0          5m22s
kube-system   coredns-7ff77c879f-q5ll2                    1/1     Running   0          5m22s
kube-system   etcd-master01.paas.com                      1/1     Running   0          5m32s
kube-system   kube-apiserver-master01.paas.com            1/1     Running   0          5m32s
kube-system   kube-controller-manager-master01.paas.com   1/1     Running   0          5m32s
kube-system   kube-proxy-th472                            1/1     Running   0          5m22s
kube-system   kube-scheduler-master01.paas.com            1/1     Running   0          5m32s
[root@master01 ~]# kubectl get node
NAME                STATUS   ROLES    AGE     VERSION
master01.paas.com   Ready    master   5m47s   v1.18.0
[root@master01 ~]#
```

此时集群状态正常

# 7 安装kubernetes-dashboard

官方部署dashboard的服务没使用nodeport，将yaml文件下载到本地，在service里添加nodeport

```shell
[root@master01 ~]# wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-rc7/aio/deploy/recommended.yaml
[root@master01 ~]# vim recommended.yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard

[root@master01 ~]# kubectl create -f recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

查看pod，service

```shell
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-dc6947fbf-869kf   1/1     Running   0          37s
kubernetes-dashboard-5d4dc8b976-sdxxt       1/1     Running   0          37s
[root@master01 ~]# kubectl get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.10.58.93    <none>        8000/TCP        44s
kubernetes-dashboard        NodePort    10.10.132.66   <none>        443:30000/TCP   44s
[root@master01 ~]#
```

我们通过浏览器来访问一下
https://192.168.1.202:30000
接上一步，看到了登录界面，需要我们配置输入token或kubeconfig，这里我们选择token，通过以下命令获取输出的token

```shell
kubectl create serviceaccount dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

拿到token在登录界面的令牌区域输入，然后点击登录
如果忘记Token，使用下面的命令
```shell
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

即可进入下图所示的主界面了

重装Dashboard
在kubernetes-dashboard.yaml所在路径下

```shell
kubectl delete -f kubernetes-dashboard.yaml
kubectl create -f kubernetes-dashboard.yaml
```

查看所有的pod运行状态
'
```shell
kubectl get pod --all-namespaces
```

查看dashboard映射的端口

```shell
kubectl -n kube-system get service kubernetes-dashboard
```

