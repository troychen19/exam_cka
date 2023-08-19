| 目錄 |
| --- |
| [建立刪除 daemonset](#建立刪除-daemonset) |
| [指定 pod 所在位置](#指定-pod-所在位置) |
| [其它控制器](#其它控制器) |

# daemonset 及其它控制器

---
**重點**  
1. 建立/刪除 daemonset
2. 指定 pod 在特定的節點
---

daemonset (簡稱 ds) 與 deployment 類似，也是 pod 控制器，差別在於 daemonset 會在所有的節點 (包括 master) 上建一個 pod。
例如 kebe-proxy 這個 pod ，每個節點有會有一個，加了新節點就是由 daemonset 自動運行一個 kube-proxy。  
daemonset 一般用在監控或日誌，每個節點運行一個，這樣可以收集所有主機的監控信息或日試。  

查看 daemonset
```bash
 kubectl get ds
```
---

## 建立刪除 daemonset

1. 建立 daemonset 需要的 yaml 檔
   ```bash
     kubectl create deployment ds1 --image=busybox --dry-run=client -o yaml -- sh -c "sleep 36000" > ds1.yaml
   ```
2. 修改 yaml 檔
   * kind 的值改為 daemonset
   * 刪除 spec 中的 replicas
   * 刪除 spec 中的 strategy
   * 刪除 status
   ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
    creationTimestamp: null
    labels:
        app: ds1
    name: ds1
    spec:
    selector:
        matchLabels:
        app: ds1
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: ds1
        spec:
        containers:
        - command:
            - sh
            - -c
            - sleep 36000
            image: busybox
            name: busybox
            resources: {}
   ```
查看除了 master 外，都建立了 pod，没有 replicas 的設定，只會在所有 node 建立一個 pod
   ```bash
    # kubectl get ds
    NAME   DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
    ds1    2         2         2       2            2           <none>          4s
   ```
刪除 ds
   ```bash
    kubectl delet ds ds1
   ```

---

## 指定 pod 所在位置

1. 在 node1 加入標籤
   ```bash
    kubectl label node node01 node=worker1
   ```
2. 修改 yaml 檔，加入 nodeSelector
   ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
    creationTimestamp: null
    labels:
        app: ds1
    name: ds1
    spec:
    selector:
        matchLabels:
        app: ds1
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: ds1
        spec:
        nodeSelector:
                node: worker1
        containers:
        - command:
            - sh
            - -c
            - sleep 36000
            image: busybox
            name: busybox
            resources: {}
   ```

---

## [其它控制器](./ReplicaSet.md)

這些控制的作用是相似的，只是在 yaml 文件的語法有些區別，總結如下：

| | api | select |
|---|---|---|
| deployment | apps/v1 | seletor:</br>matchLabels: |
| daemonset | apps/v1 | seletor:</br>matchLabels: |
| ReplicationController | v1 | selector: |
| ReplicaSet | apps/v1 |seletor:</br>matchLabels:|