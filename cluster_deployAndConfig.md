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
```bash
kubeadm init --kubernetes-version=v1.26.5 \
--upload-certs \
--pod-network-cidr=10.244.0.0/16
```
參數：
 * --image-repository： container image 倉庫位置
 * --kubernetes-version： k8s 安裝版本
 * --pod-network-cidr： pod 使用的網段， ex: --pod-network-cidr=10.210.0.0/16
 * --service-cidr:  service 使用網段，ex: --svc-network-cidr=10.215.0.0/16
 * --control-plane-endpoint：如果打算建立HA(High Availability)集群，則需要設定這個，可以輸入Load balancer的DNS名稱或是IP，HA的部分會在未來說明。

## 2. 配置 worker

用初始化產生的 token 與指令加入 worker
```bash
kubeadm join 192.168.153.102 --token <token value>
```

## 3. 安裝 pod network

* get yaml

  **Calico**
  ```bash
  curl -O -L https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
  ```
  **flannel**
  ```bash
  curl -O -L https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  ```

* apply

  **Calico**
  ```bash
  kubectl apply -f ./calico.yaml
  ```
  **flannel**
  ```bash
  kubectl apply -f ./kube-flannel.yml
  ```

## 4. 刪除節點及重新加入

**移除 node02**
- 將 node02 設成維護模式
   ```bash
   kubectl drain node02 --delete-emptydir-data --force --ignore-daemonsets
   ```
- 刪除節點
  vag ```bash
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
在 github 的專案 [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server), 下載位置： [metrics-servcer](https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml)

- 安裝 metric-server
下載 yaml
```bash
curl -O -L https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

修改 CA
```bash
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        # 新增相信憑證
        - --kubelet-insecure-tls=true
        image: registry.k8s.io/metrics-server/metrics-server:v0.6.3

```

安裝 metrics-server
```bash
kubectl apply -f components.yaml
```

- 使用方式 

1. 查看節點負載
   ```bash
   kubectl top nodes --use-protocol-buffers
   ```
2. 查看 POD 的負載
   ```bash
   kubectl top pods -n kub-system --use-protocol-buffers
   ```

# 命名空間 namespace

命名空間用來區分不同工能的 POD 群組，基本指令如下：
1. 查看目前有多少 namespace
   ```bash
   kubectl get ns
   ```
2. 創建一個新的 namespace
   ```bash
   kubectl create ns ns1
   ```

# 升級 kubernetes
先升 master，再升 worker, 先升 kubeadm 再用 kubeadm upgrade 升元件，最後再升 kubectl 與 kubelet

1. 查看目前版本
   ```bash
     kubectl get nodes
   ```
2. 確定 yum source 中可用的 kubeadm 版本
   ```bash
     apt-cache madison kubeadm
     apt-cache policy kubeadm
   ```
3. 升級 kubeadm
   
   不管升級 master 或 worker，都要先升 kubeadm，以下使用 1.26 升到 1.27 為例
   ```bash
      apt-mark unhold kubeadm && \
      apt-get update && apt-get install -y kubeadm=1.27.4-00 && \
      apt-mark hold kubeadm
   ```
   安裝完成後驗證 kubeadm 的版本
   ```bash
     kubeadm version
   ```
4. 透過 kubeadm upgrade plan 查看群集是否需要升級
   ```bash
     kubeadm upgrade plan
   ```
5. 把 master 設置為維護模式，並清空 pod
   ```bash
     kubectl drain master --ignore-daemonsets
   ```
6. 升級 master 上各組件
   ```bash
     kubeadm upgrade apply v1.27.3
   ```
7. 升級 master 上的 kubelet 和 kubectl

   安裝 v1.27.4 的 kubelet 與 kubectl
   ```bash
     apt-mark unhold kubelet kubectl && \
     apt-get update && apt-get install -y kubelet=1.27.4-00 kubectl=1.27.4-00 && \
     apt-mark hold kubelet kubectl  
   ```
8. 重啟 kubelet 與 kubectl
   ```bash
      systemctl deamon-reload
      systemctl restart kubelet
   ```
9. 完成後取消維護模式
  ```bash
    kubectl uncordon master
  ```
10. 升 worker
    * 升 kubeadm 到 1.27.4
    * 設定維護模式
    * 升級元件 kubeadm upgrade node
    * 升級 kubelet 與 kubectl
    * 取消維護模式

