1. 镜像文件 [下载链接](https://github.com/bertreyking/repos/releases)
```
docker load -i dashboard-v2.0.0-beta8.tar.gz
docker load -i metrics-scraper-v1.0.2.tar.gz
```
2. dashboard_yaml文件
```
- yaml 中已包含service account 及 clusterrole 等资源，无需重新创建
- 原文件不支持nodePort，仅做了支持nodePort访问的修改
```

[k8s for dashboard](https://github.com/kubernetes/dashboard)

[参考blog](https://www.replex.io/blog/how-to-install-access-and-add-heapster-metrics-to-the-kubernetes-dashboard)
