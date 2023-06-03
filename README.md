# 1. 測試

## 1.1 使用環境
在 Ubuntu 20.04 下建 k8s 1.26.5 的測試環境 
使用 containerd 做為 k8s 的容器，並使用 docker 管理 image

## 1.2 初始化並加入 worker node
```shell
sudo kubeadm init --control-plan-endpoint="master"
```
init 參數說明

## 1.3 使用 Calico 並確認所有的 ndoe ready


* download yaml
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
```
* apply
```
kubectl apply -f ./calico.yaml
```

# 2. 容器模擬題


