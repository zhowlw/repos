# deploy_k8s_masterNode

- 基础环境规划
```
1.节点配置 
k8s-master01    4c/8g/80g_os、100g_docker_volume     10.6.21.x root/xxxxxx  ansible_node_v2.4.2.0
k8s-master02    4c/8g/80g_os、100g_docker_volume     10.6.21.x root/xxxxxx
2.系统优化
#关闭selinux安装机制
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
#禁用swap分区(若节点有其他应用需要，可以在kubeadm init 初始化集群时加上忽略相关参数即可)
swapoff -a
#系统内核优化
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

- 配置ansible运维工具
```
1.安装ansible 
yum isntall -d ansible && yum install -y ansible

2.节点间配置互信
[root@k8s-master01 ~]# ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:9FXoOaiwRSFzFvnm2z1GrQioCqebmgpl8awXg0yPqD4 root@k8s-master01
The key's randomart image is:
+---[RSA 2048]----+
|      o =+   ..  |
|       =o   ..   |
|  o    ... o..   |
| + B  ....+.+    |
|. * *  +S=.  . . |
|.o . o. o o   . .|
|o o o  .   + + . |
|oE *  .   . o =  |
|=o=...       . . |
+----[SHA256]-----+
[root@k8s-master01 ~]# 

[root@k8s-master01 ~]# ssh-copy-id -i .ssh/id_rsa.pub root@10.6.203.61
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: ".ssh/id_rsa.pub"
The authenticity of host '10.6.203.61 (10.6.203.61)' can't be established.
ECDSA key fingerprint is SHA256:pCTuIzcK0DBf8g0rnDDZi9RycoS1d/iQoAyBQjrV5g4.
ECDSA key fingerprint is MD5:ab:26:61:6e:00:32:20:6b:42:3b:af:50:4e:b5:f2:8f.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.6.203.61's password: 
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'root@10.6.203.61'"
and check to make sure that only the key(s) you wanted were added.

3.配置ansible主机组
vi /etc/ansible/hosts
[k8s]
10.6.203.[60:61]

4.验证ansible工作状态
ansible all -m ping 
10.6.203.60 | UNREACHABLE! => {
    "changed": false, 
    "msg": "Failed to connect to the host via ssh: Warning: Permanently added '10.6.203.60' (ECDSA) to the list of known hosts.\r\nPermission denied (publickey,gssapi-keyex,gssapi-with-mic,password).\r\n", 
    "unreachable": true
}
如果遇到这种报错，请将ansible节点所所生成的pub密钥追加到.ssh/目录下的authorized_keys文件中即可解决(ansible执行命令是使用python库调用节点的ssh服务，若此时不能免密登陆，就会报这个错误),但是后续使用还是会ask，把ansible节点改为StrictHostKeyChecking no
(默认为ask)
ansible all -m ping 
10.6.203.60 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
10.6.203.61 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```
- 安装docker
```
1.安装devicemapper相关工具
yum install -y yum-utils device-mapper-persistent-data lvm* 
2.下载docker_repo文件
yum install -y wget*
cd /etc/yum.repos.d/ && wget https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --disable docker-ce-stable
yum-config-manager --enable docker-ce-edge
3.检查docker-ce-edge中docker的版本列表
yum list docker-ce --showduplicates | sort -r
4.安装docker
yum install -y docker-ce-18.06.3.ce-3.el7
5.配置lv分区，docker使用overlay2fs
pvcreate /dev/sdb
vgcreate vgdocker /dev/sdb
lvcreate -n lvdocker -L 99G vgdocker
mkfs.xfs /dev/mapper/vgdocker-lvdocker (xfs_growfs /dev/mapper/vgdocker-lvdocker  看情况，一般扩容才会用到)
echo "/dev/mapper/vgdocker-lvdocker         /var/lib/docker/          xfs     defaults        0 0" >> /etc/fstab
echo "mount /dev/mapper/vgdocker-lvdocker /var/lib/docker/" >> /etc/rc.local && chmod +x /etc/rc.d/rc.local
mkdir -p /var/lib/docker && mkdir -p /etc/docker
mount /dev/mapper/vgdocker-lvdocker /var/lib/docker/
6.配置&启动docker
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
systemctl start docker && systemctl status docker
```

- 安装kubeadm
```
1.定义repo仓库文件
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
2.安装kubeadm、kubelet、kubectl
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
- 从dockerhub下载k8s组件镜像文件(docker将k8s_image镜像到了dockerHub中)

[k8s_images](https://hub.docker.com/u/mirrorgooglecontainers/)
```
1. 下载镜像文件
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.16.0
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.16.0
docker pull mirrorgooglecontainers/pause:3.1
docker pull mirrorgooglecontainers/etcd:3.3.15-0
docker pull coredns/coredns:1.6.2
2.启动一个registry仓库，方便后续节点接入
docker run -itd --name registry -p 5000:5000 registry:latest
3.更改镜像tag并push到registry
docker tag k8s.gcr.io/kube-apiserver:v1.16.0          10.6.203.60:5000/kube-apiserver:v1.16.0   
docker tag k8s.gcr.io/kube-proxy:v1.16.0              10.6.203.60:5000/kube-proxy:v1.16.0
docker tag k8s.gcr.io/kube-controller-manager:v1.16.0 10.6.203.60:5000/kube-controller-manager:v1.16.0
docker tag k8s.gcr.io/kube-scheduler:v1.16.0          10.6.203.60:5000/kube-scheduler:v1.16.0 
docker tag k8s.gcr.io/etcd:3.3.15-0                   10.6.203.60:5000/etcd:3.3.15-0 
docker tag k8s.gcr.io/coredns:1.6.2                   10.6.203.60:5000/coredns:1.6.2
docker tag k8s.gcr.io/pause:3.1                       10.6.203.60:5000/pause:3.1 

docker push 10.6.203.60:5000/kube-apiserver:v1.16.0   
docker push 10.6.203.60:5000/kube-proxy:v1.16.0
docker push 10.6.203.60:5000/kube-controller-manager:v1.16.0
docker push 10.6.203.60:5000/kube-scheduler:v1.16.0 
docker push 10.6.203.60:5000/etcd:3.3.15-0 
docker push 10.6.203.60:5000/coredns:1.6.2
docker push 10.6.203.60:5000/pause:3.1 
```
- kubeadm 初始化集群
```
1.打印默认的cluster配置文件
kubeadm config print init-defaults
2.基础默认修改cluster配置文件
kubeadm init --config k8s-cluster001.yaml
3.将imageRepository更改为刚刚创建的registry
imageRepository: 10.6.203.60:5000
[root@k8s-master01 ~]# kubeadm init --config k8s-cluster001.yaml
[init] Using Kubernetes version: v1.16.0
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "k8s-master01" could not be reached
        [WARNING Hostname]: hostname "k8s-master01": lookup k8s-master01 on 192.168.1.29:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.6.203.60]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master01 localhost] and IPs [10.6.203.60 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master01 localhost] and IPs [10.6.203.60 127.0.0.1 ::1]
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
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 34.503654 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.16" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: abcdef.0123456789abcdef
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
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

kubeadm join 10.6.203.60:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:3df3bcde7bddf019fced3d8a726c458ae09c4d8bd0a87585cd740ff946bf1830 
```
- 部署calico网络
[k8s_network](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
[calico_install](https://docs.projectcalico.org/v3.9/getting-started/kubernetes/)
```
[root@k8s-master01 ~]# kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
```

- 查看全部namespaces下的资源
```
[root@k8s-master01 ~]# kubectl get all --all-namespaces 
NAMESPACE     NAME                                          READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-dc6cb64cb-s5l6d   1/1     Running   0          168m
kube-system   pod/calico-node-x5s4f                         1/1     Running   0          168m
kube-system   pod/coredns-765bb46f58-5pmvm                  1/1     Running   0          175m
kube-system   pod/coredns-765bb46f58-qgrzj                  1/1     Running   0          175m
kube-system   pod/etcd-k8s-master01                         1/1     Running   0          174m
kube-system   pod/kube-apiserver-k8s-master01               1/1     Running   0          174m
kube-system   pod/kube-controller-manager-k8s-master01      1/1     Running   0          174m
kube-system   pod/kube-proxy-kxh22                          1/1     Running   0          175m
kube-system   pod/kube-scheduler-k8s-master01               1/1     Running   0          174m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  175m
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   175m

NAMESPACE     NAME                         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
kube-system   daemonset.apps/calico-node   1         1         1       1            1           beta.kubernetes.io/os=linux   168m
kube-system   daemonset.apps/kube-proxy    1         1         1       1            1           beta.kubernetes.io/os=linux   175m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           168m
kube-system   deployment.apps/coredns                   2/2     2            2           175m

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-dc6cb64cb   1         1         1       168m
kube-system   replicaset.apps/coredns-765bb46f58                  2         2         2       175m
[root@k8s-master01 ~]# kubectl get nodes 
NAME           STATUS   ROLES    AGE    VERSION
k8s-master01   Ready    master   176m   v1.16.2
```
