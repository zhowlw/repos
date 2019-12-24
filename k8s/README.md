- [x] master 节点初始化配置及安装
- [x] worker 节点初始化配置及接入
- [x] dashboard 配置 （部署后，chrome无法打开，火狐可以，下附解决方法）
- [x] kubectl 命令tab补全

1. kubectl 命令tab补全
```
rpm -qa | grep bash-completion
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```
2. dashboard （chrome 无法打开 dashboard）[参考链接]（https://github.com/kubernetes/dashboard/issues/2947）
```
原因： 浏览器配置不允许使用自签名证书，因为yaml所创建dashboard的secrets中没有证书信息，最终导致无法正常访问
# mkdir /certs

# openssl req -nodes -newkey rsa:2048 -keyout /certs/dashboard.key -out /certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"

# openssl x509 -req -sha256 -days 365 -in /certs/dashboard.csr -signkey /certs/dashboard.key -out /certs/dashboard.crt

# kubectl create secret generic kubernetes-dashboard-certs --from-file=/certs -n kubernetes-dashboard

# kubectl describe secret kubernetes-dashboard-certs -n kubernetes-dashboard

# kubectl describe secret kubernetes-dashboard-certs -n kubernetes-dashboard                                         
Name:         kubernetes-dashboard-certs
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  <none>
Type:  Opaque
Data
====
dashboard.csr:  907 bytes
dashboard.key:  1704 bytes
dashboard.crt:  1005 bytes
```
