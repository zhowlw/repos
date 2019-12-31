
# yaml文件示例（新增每个资源类型的具体输出）

- Cronjob 输出：
```
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
[root@k8s-master01 yaml]# kubectl logs -f centos-cronjob-echo-1577799540-xl8ht -n test-tenant 
This is a Test cronjob！
```
