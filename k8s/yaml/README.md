
# yaml文件示例（新增每个资源类型的具体输出）

1. Cronjob 输出：
```
- 查看job数
[root@k8s-master01 yaml]# kubectl get jobs -n test-tenant                        
NAME                             COMPLETIONS   DURATION   AGE
centos-cronjob-echo-1577800320   4/4           10s        10m
centos-cronjob-echo-1577800380   4/4           8s         9m8s
centos-cronjob-echo-1577800440   4/4           8s         8m8s
- 查看pod数
[root@k8s-master01 yaml]# kubectl get pods -n test-tenant  | grep cronjob
centos-cronjob-echo-1577799420-2rtcd   0/1     Completed   0          2m20s
centos-cronjob-echo-1577799420-8l4sl   0/1     Completed   0          2m26s
centos-cronjob-echo-1577799420-g55fb   0/1     Completed   0          2m20s
centos-cronjob-echo-1577799420-l94bg   0/1     Completed   0          2m26s
centos-cronjob-echo-1577799480-4tscg   0/1     Completed   0          82s
centos-cronjob-echo-1577799480-n6q8f   0/1     Completed   0          86s
centos-cronjob-echo-1577799480-swf9s   0/1     Completed   0          82s
centos-cronjob-echo-1577799480-x8bc5   0/1     Completed   0          86s
centos-cronjob-echo-1577799540-f7qsx   0/1     Completed   0          22s
centos-cronjob-echo-1577799540-jwc2p   0/1     Completed   0          22s
centos-cronjob-echo-1577799540-x26bq   0/1     Completed   0          26s
centos-cronjob-echo-1577799540-xl8ht   0/1     Completed   0          26s
- 查看pod_log
[root@k8s-master01 yaml]# kubectl logs -f centos-cronjob-echo-1577799540-xl8ht -n test-tenant 
This is a Test cronjob！
- 查看master节点是否有污点
[root@k8s-master01 yaml]# kubectl get nodes k8s-master01 --show-labels            
NAME           STATUS   ROLES    AGE   VERSION   LABELS
k8s-master01   Ready    master   38d   v1.16.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8s-master01,kubernetes.io/os=linux,node-role.kubernetes.io/master=
- 打污点
[root@k8s-master01 yaml]# kubectl taint node k8s-master01 node-role.kubernetes.io=master:NoSchedule
node/k8s-master01 tainted
- 取消污点
[root@k8s-master01 yaml]# kubectl taint node k8s-master01 node-role.kubernetes.io=master:NoSchedule-
node/k8s-master01 untainted

- completions: 4 job完成总数
- parallelism: 2 job的并发数
- successfulJobsHistoryLimit：保留已完成job数量，默认是3
- failedJobsHistoryLimit: 1 保留失败job数量，默认是1
- 4个节点、1个master节点(默认有污点)、get pods 正好是12个

```
