
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
pod/dashboard-metrics-scraper-b76b67fc8-c9cxp   1/1     Running   0          2m28s
pod/kubernetes-dashboard-795f7465f5-b85qz       1/1     Running   0          2m28s
NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/dashboard-metrics-scraper   ClusterIP   10.100.102.36   <none>        8000/TCP   2m29s
service/kubernetes-dashboard        ClusterIP   10.103.81.117   <none>        443/TCP    2m30s
NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2m29s
deployment.apps/kubernetes-dashboard        1/1     1            1           2m29s
NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/dashboard-metrics-scraper-b76b67fc8   1         1         1       2m29s
replicaset.apps/kubernetes-dashboard-795f7465f5       1         1         1       2m29s
```
3. 启动临时proxy
```
[root@k8s-master01 dashboard]# kubectl proxy
Starting to serve on 127.0.0.1:8001

[root@k8s-master01 ~]# netstat -anltp | grep 8001
tcp        0      0 127.0.0.1:8001          0.0.0.0:*               LISTEN      2857/kubectl 
```

4. 验证
```
curl 
```
