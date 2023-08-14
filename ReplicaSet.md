# 一、ReplicaSt
Replication 可讓部署的 POD 具有以下的特點：
* High Availability：POD 異常，仍然可以正常提供服務
* Load Balancing & Scaling： 可分散負載並隨時調整 POD 的數量

實現的方式有兩種 Replication Controller 與 ReplicaSet，其中 Rplication Controller 被 RepliceSet 取代。
主要差異如下：
* Seletor：ReplicaSet 是 set-based 與 ReplicationController 是 equality-based，簡單來説 set-based就是 tier in (frontend, backend)，equality-based 就是 tier=frontend，前者靈活了不少。
* Update：ReplicationController 只支援 rolling-update 指令更新，而 ReplicaSet 可以透過 Deployment 做出更靈活的更新方式，像是 rollout。

比較表
| Replication Controller | Replica Set |
| --- | --- |
| The Replication Controller is the original from of replication in Kubernetes | ReplicaSets are a higher-level API that gives the ability to easily run multiple instance of a given pod |
| The Replication Controller uses __equality-based selectors__ to manage the pods.| ReplicaSet Controller user __set-baed selectors__ to manage the pods.|
|The rolling-update command works with Replication Controllers| The rolling-update command won't work with ReplicaSets.|
|Replica Controller is deprecated and replace by ReplicaSet| Deployments are recommended over RplicaSets.|

**註** set-based 與 equality-based 的差異

## 1.1 Replication Controller

以下為使用 replicate controller 的範例，YAML 檔說明如下：
1. apiVersion 為 v1 
2. kind 為 ReplicationController
3. 需要定義 template
4. metadata 的 lables 要與 template 中的 metadata 定義的 labels 一致
5. replicas 定義擴展的數量

Ex：
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-app
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-app
    spec:
      container:
      - name: nginx-container
        image: nginx
replicas: 3
```

## 2. ReplicaSet

1. apiVersion 為 app/v1 
2. kind 為 ReplicaSet
3. 需要設定 seletor
4. selector 定義的 matchLabels 要與 label 的 type 相同

Ex:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp
      label:
        app: myapp
        type: front-end
    spce:
      containers:
      - name: nginx-container
        image: nginx
replicas: 3
selector:
  matchLabels:
    type: front-end
```

## 3. 操作指令

1. 顯示 replicaset
```bash
 kubectl get replicaset -n default

 kubectl get rs
```
2. 建立 replicaset
```bash
  kubectl creat -f replicate-definetion.yam
```
3. 檢視詳細資料
```bash
  kubectl describe replicaset <replicaset name>
```
4. 刪除 replicaset
```bash
  kubectl delete replicaset <replicaset name>

  kubectl delete -f replicaset.yaml
```
5. 修改已存在的 replica
```bash
  kubectl edit repliacset <replicaset name>
```
6. 指令修改 replica 數量
```bash
  kubectl scale --replicas=<number> rs/<replicaset name>
```