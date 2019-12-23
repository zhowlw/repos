
1. 下载镜像文件&push 到registry中
```
docker pull kubernetesui/dashboard:v2.0.0-beta8
docker pull kubernetesui/metrics-scraper:v1.0.2
docker tag kubernetesui/dashboard:v2.0.0-beta8 10.6.203.60:5000/dashboard:v2.0.0-beta8
docker tag kubernetesui/metrics-scraper:v1.0.2 10.6.203.60:5000/metrics-scraper:v1.0.2
docker push 10.6.203.60:5000/dashboard:v2.0.0-beta8
docker push 10.6.203.60:5000/metrics-scraper:v1.0.2
sed -i 's#kubernetesui/dashboard:v2.0.0-beta8#10.6.203.60:5000/dashboard:v2.0.0-beta8#g'  recommended.yaml 
sed -i 's#kubernetesui/metrics-scraper:v1.0.1#10.6.203.60:5000/metrics-scraper:v1.0.2#g'  recommended.yaml 
```
2. 部署dashboard资源
```
[root@k8s-master01 dashboard]# kubectl apply -f recommended.yaml 
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
[root@k8s-master01 dashboard]# 

[root@k8s-master01 dashboard]# kubectl get all -n kubernetes-dashboard
NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-b76b67fc8-c9cxp   1/1     Running   0          155m
pod/kubernetes-dashboard-795f7465f5-79bw5       1/1     Running   0          117m
NAME                                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/dashboard-metrics-scraper   NodePort   10.100.102.36   <none>        8000:30642/TCP   155m
service/kubernetes-dashboard        NodePort   10.103.81.117   <none>        443:30010/TCP    155m
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           155m
deployment.apps/kubernetes-dashboard        1/1     1            1           155m
NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-b76b67fc8   1         1         1       155m
replicaset.apps/kubernetes-dashboard-795f7465f5       1         1         1       155m
```
3. 在default中创建service account 并绑定给cluster-admin 管理员角色
```
kubectl create serviceaccount test01

kubectl create clusterrolebinding test01 --clusterrole=cluster-admin --serviceaccount=default:test01

kubectl get secrets

kubectl describe secrets xxxxx -n default     ####将tocken复制保存以备登陆时使用
```
4. 验证 https://nodeIP:30010 选择tocken、输入tocken即可
