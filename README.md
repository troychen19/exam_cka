# 使用環境
在 Ubuntu 20.04 下建 k8s 1.26.5 的測試環境 
使用 containerd + k8s，並使用 docker 管理 image

# 初始化
```shell
sudo kubeadm init --control-plan-endpoint="master"
```