# Schaudle

當部署 pod 的時候，kubernetes 會自動指定 nodeName 在部署檔，因此我們也可以手動指定要部署的節點。
| 目錄 |
| --- |
| [Labels 與 Selector](#Labels-與-Selector) |
| [節點 taint 與 pod 的 tolerations ](#節點-taint-與-pod-的-tolerations ) |
| [Node Selector](#node-selector) |
| [Node Affinity](#node-affinity)|

## Labels 與 Selector


在 pod 定義 lablel ，讓 ReplicaS 與 Service  用來識別要傳送的 pod，每當擴展 repilac 新產生的 pod label 會與 relplica 相同
使用 service 或 replica 需要先在 pod 定義 label
語法:以下定義兩個 label app=App1, function=Front-end
```yaml
 apiVersion: v1
 kind: Pod
 metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
 spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
```
如需要使用 replicas 設定如下：
各層的 labels 設定都要一致，同時在 selector 設定 match app=App1
```yaml
 apiVersion: apps/v1
 kind: ReplicaSet
 metadata:
   name: simple-webapp
   labels:
     app: App1
     function: Front-end
 spec:
  replicas: 3
  selector:
    matchLabels:
     app: App1
 template:
   metadata:
     labels:
       app: App1
       function: Front-end
   spec:
     containers:
     - name: simple-webapp
       image: simple-webapp   
```

## 節點 taint 與 pod 的 tolerations 
k8s doc:(Concepts/Scheduling/Taints and Tolerations)

**[重點]** 給節點設置/刪除 taint，設置 operator 的值為 Equal，以及設置 operator 的值為 Exists

當建立 pod 雖然 master 也是 ready，但 pod 並不會部署到 master。因為 master 有設置 taint (污點)。
當我們給某節點設置了 taint，只有那些設置了 tolerations 的 pod 可以運行在這些節點。

1. 查看 taint 的語法
```bash
  kubectl describe ndoes node01 | grep -E '(Roles|Taints)'
```
2. 設定節點 taint
設定節點 taint key=value 的語法為 key值=value值:effect ，一般 effect 為設為 NoSchedule 
    * 設定所有節點
    ```bash
      kubectl taint nodes -all key1=all:NoSchedule
    ```
    * 設定指定節點
    ```bash
      kubectl tain nodes node01 key1=node1:NoSchedule
    ```
3. 要將 pod 部署在設有 taint 的 node
需要在 yaml 中加入以下的設定，以下的設定表示 pod 會被部署到 taint 標註為 key1=node1 的節點
```yaml
tolerations:
- key: "key1" # key 的值
  operator: "Equal"
  value: "node1" # value 的值
```
operator 的值有兩種 Equal 和 Exists， Equal 的 key 與 value 需要和 node 相同，Exists 則不需指定 value 值
使用 Exists 時，設定 node 與 pods 設定檔都不能設定 value 

## Node Selector
設定 node label 的 key=value 並在 pod 指定 nodeSelector 可以讓 pod 部署在指定的 node
```bash
kubectl label nodes node-1 size=Large
```
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 nodeSelector:
  size: Large
```

## Node Affinity 
k8s doc:(concepts/scheduling/Assing Pod to Nodes)

Node Affinity 會確保 pod 部署在指定的 node，與 nodeSelector 不同的是，Node Affinity 可指定不同的操作模式。
Affinity 的種類分四個，差別在於強制與非強制，執行中受影響的 pod 是否搬移：
* requiredDuringSchedulingIngnoredDugingExecution
* preferredDuringSchedulingIngnoredDuringExecution
* requiredDuringSchedulingDuringExection
* preferredDuringSchedulingDuringExection

operators 的種類有：
* In：在 key=value 的範圍
* NotIn
* Exists：只要 key 存在就符合
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values: 
            - Large
            - Medium
```

## Taints vs Affinity
Taints and Tolerations 不保證一定按設定，因此可以合併 Node Affinity 來使用

## Resource Requirement and Limits 
k8s doc: (concepts/configuration/Resource Management)
CPU 1: 1 AWS vCPU, 1GCP 
Memory: 268M, 1G, 1 K
Memory 與記憶體不同之處在於，記憶體超出使用設定，該 pod 會發出 OOM 錯誤後停止

LimitRange 使用在 namespace

使用上可在 pod 設 resource 或使用 LimitRange 對一個 namespace 限制

ResourceQuota 

## DaemonSet
=======

