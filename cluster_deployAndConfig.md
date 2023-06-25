# 前言

*部署 kubernetes 集群*

考試大網： 了解 kubernetes 的架構，並部署 kubernetes 集群
重點：
1. 使用 kubeadm 部署 kubernetes 集群
2. 添加及刪除 worker
3. 查看 pod 與節點負載
4. 了解及管理命名空間

---

**master 與 worker 上運行組件及作用**

| 組件名稱 | node | 作用 |
|---|---|---|
| kubectl | naster | 命令工具，用來建立、刪除 pod, service ... |
| api-server | master | 接收用戶的請求 |
| scheduler | master | 資源調度，當建立 pod 時，會判斷 pod 被放到哪一個 worker |
| controller-manager | master | k8s 的管家，監測節點狀態、pod 數量... |
| kubelet | worker | 在所有的 master 與 worker 上運行，是一個代理，接受 master 分配過來的任務，並把節點信息回覆給 master 上的 api-server |
| kube-proxy | worker| 在所有的 master 與 worker 上運行，用於從 service 傳送過來的 request 轉送到 pod，模式有 iptables 與 ipvs |
| calico | worker | 讓節點中的 pod 可以相互通信 |

---

**快速指令環境設定**




---

# 一 安裝部署 cluster

## 1. 安裝 master
```shell
kubeadm init --kubernetes-version=v1.26.5
```
參數：
 * --image-repository： container image 倉庫位置
 * --kubernetes-version： k8s 安裝版本
 * --pod-network-cidr： pod 使用的網段
 * --control-plane-endpoint：如果打算建立HA(High Availability)集群，則需要設定這個，可以輸入Load balancer的DNS名稱或是IP，HA的部分會在未來說明。

## 2. 配置 worker

用初始化產生的 token 與指令加入 worker
```bash
kubeadm join 192.168.153.102 --token <token value>
```

## 3. 安裝 calico
* get yaml
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
```
* apply
```
kubectl apply -f ./calico.yaml
```

## 4. 刪除節點及重新加入

**移除 node02**
- 將 node02 設成維護模式
   ```bash
   kubectl drain node02 --delete-emptydir-data --force --ignore-daemonsets
   ```
- 刪除節點
   ```bash
   kubectl delete node node02
   ```
- 檢查已刪除
   ```bash
   kubectl get nodes
   ```

- 清空節點上配置 (在 node02 上操作)
   ```bash
   kubeadm reset
   ```
**加入 node3**
- 檢查目前的 token (超過 24H 要重新產生)
   ```bash
   kubeadm token list
   ```
- 重新產生 token
  ```bash
   kubeadm token create --print-join-command
  ```
- 複製上述指令產生的 join 指令加入 node2

# 常見命令

# metric-server 監控 pod 與節點負載
使用 `metrics-server-amd64`，下載位置： [metrics-servcer](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)

- 安裝 metric-server
在 github 的專案 [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
- 修改 CA

- 使用方式 


# 命名空間 namespace

# 升級 kubernetes
