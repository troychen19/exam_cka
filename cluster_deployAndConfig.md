# 一 部署 cluster

1. 安裝 master
```shell
kubeadm init --kubernetes.version=v1.26.5 --pod-netowrk=192.168.0.0/16
```
參數：
 * --image-repository： container image 倉庫位置
 * --kubernetes.version： k8s 安裝版本
 * --pod-network： pod 使用的網段
 * --control-plane-endpoint：如果打算建立HA(High Availability)集群，則需要設定這個，可以輸入Load balancer的DNS名稱或是IP，HA的部分會在未來說明。

2. 配置 worker

用初始化產生的 token 與指令加入 worker
```shell
kubeadm join 192.168.153.102 --token <token value>
```

3. 安裝 calico
4. 刪除節點及重新加入


5. 常見命令
6. metric-server 監控 pod 與節點負載
7. 命名空間 namespace

# 升級 kubernetes
