- [x] master 节点初始化配置及安装
- [x] worker 节点初始化配置及接入
- [x] dashboard 配置
- [x] kubectl 命令tab补全

1. kubectl 命令tab补全
```
rpm -qa | grep bash-completion
yum install -y bash-completion
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```
