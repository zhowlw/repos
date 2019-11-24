- 安装kubelet、kubeadm
```
1. 安装依赖软件
# yum install -y conntrack socat
2. 安装kubelet、kubeadm、kubectl(可选)
# yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
- 接入计算节点
```
1.master节点生成新的接入tocken
# kubeadm token create --print-join-command
2.worker节点执行
# kubeadm join 10.6.203.60:6443 --token 9fzhfo.njjsd9yuo6tjml7t     --discovery-token-ca-cert-hash sha256:5a580d0a6f8d17b69530c9ae782f35631d93449f13f0350de5523a7d06b2c126

[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "k8s-master03" could not be reached
        [WARNING Hostname]: hostname "k8s-master03": lookup k8s-master03 on 192.168.1.29:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.16" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
3. 查看节点/pod状态
# kubectl get nodes 
NAME           STATUS     ROLES    AGE     VERSION
k8s-master01   Ready      master   6h59m   v1.16.2
k8s-master03   NotReady   <none>   4m46s   v1.16.3 （尴尬竟然不是一个版本）
# kubectl get pods --all-namespaces (查看因为镜像无法pull成功导致异常)
NAMESPACE     NAME                                      READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-dc6cb64cb-s5l6d   1/1     Running             0          6h52m
kube-system   calico-node-x44px                         0/1     Init:0/3            0          4m55s
kube-system   calico-node-x5s4f                         1/1     Running             0          6h52m
kube-system   coredns-765bb46f58-5pmvm                  1/1     Running             0          6h59m
kube-system   coredns-765bb46f58-qgrzj                  1/1     Running             0          6h59m
kube-system   etcd-k8s-master01                         1/1     Running             0          6h58m
kube-system   kube-apiserver-k8s-master01               1/1     Running             0          6h58m
kube-system   kube-controller-manager-k8s-master01      1/1     Running             0          6h58m
kube-system   kube-proxy-7gdgg                          0/1     ContainerCreating   0          4m55s
kube-system   kube-proxy-kxh22                          1/1     Running             0          6h59m
kube-system   kube-scheduler-k8s-master01               1/1     Running             0          6h58m
4. 最后检查
# kubectl get nodes 
NAME           STATUS   ROLES    AGE     VERSION
k8s-master01   Ready    master   7h45m   v1.16.2
k8s-master03   Ready    <none>   50m     v1.16.3
# kubectl get pods --all-namespaces -o wide 
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-dc6cb64cb-s5l6d   1/1     Running   0          7h38m   192.168.32.129   k8s-master01   <none>           <none>
kube-system   calico-node-chkqq                         0/1     Running   0          7s      10.6.203.60      k8s-master01   <none>           <none>
kube-system   calico-node-clz8t                         1/1     Running   0          2m41s   10.6.203.62      k8s-master03   <none>           <none>
kube-system   coredns-765bb46f58-5pmvm                  1/1     Running   0          7h45m   192.168.32.131   k8s-master01   <none>           <none>
kube-system   coredns-765bb46f58-qgrzj                  1/1     Running   0          7h45m   192.168.32.130   k8s-master01   <none>           <none>
kube-system   etcd-k8s-master01                         1/1     Running   0          7h44m   10.6.203.60      k8s-master01   <none>           <none>
kube-system   kube-apiserver-k8s-master01               1/1     Running   0          7h44m   10.6.203.60      k8s-master01   <none>           <none>
kube-system   kube-controller-manager-k8s-master01      1/1     Running   0          7h44m   10.6.203.60      k8s-master01   <none>           <none>
kube-system   kube-proxy-6wrqq                          1/1     Running   0          31m     10.6.203.62      k8s-master03   <none>           <none>
kube-system   kube-proxy-kxh22                          1/1     Running   0          7h45m   10.6.203.60      k8s-master01   <none>           <none>
kube-system   kube-scheduler-k8s-master01               1/1     Running   0          7h44m   10.6.203.60      k8s-master01   <none>           <none>
```
