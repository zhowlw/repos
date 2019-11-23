# deploy_k8s_plan

- 基础环境规划
```
1.节点配置 
k8s-master01    4c/8g/80g_os、100g_docker_volume     10.6.203.60 root/maweibing  ansible_node_v2.4.2.0
k8s-master02    4c/8g/80g_os、100g_docker_volume     10.6.203.61 root/maweibing

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
10.6.203.61 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
10.6.203.60 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

- 安装docker
```
1.安装devicemapper相关工具
yum install -y device-mapper-persistent-data lvm2
2.下载docker_repo文件
wget https://download.docker.com/linux/centos/docker-ce.repo
3.检查docker-ce-edge中docker的版本列表
yum list docker-ce --showduplicates | sort -r
4.安装docker
yum install -y docker-ce-docker-ce-18.06.3.ce-3.el7
5.配置lv分区，docker使用overlay2fs
pvcreate /dev/sdb
vgcreate vgdocker /dev/sdb
lvcreate -n lvdocker -L 99G vgdocker
mkfs.xfs /dev/mapper/vgdocker-lvdocker (xfs_growfs /dev/mapper/vgdocker-lvdocker  看情况，一般扩容才会用到)
echo "/dev/mapper/vgdocker-lvdocker                   xfs     defaults        0 0" >> /etc/fstab
echo "mount /dev/mapper/vgdocker-lvdocker /var/lib/docker/" >> /etc/rc.local && chmod +x /etc/rc.d/rc.local
6.启动docker
systemctl start docker && systemctl status docker
```

- 安装kubeadm