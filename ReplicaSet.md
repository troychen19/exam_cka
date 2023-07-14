# ReplicaSt
Replication 可讓部署的 POD 具有以下的特點：
* High Availability：POD 異常，仍然可以正常提供服務
* Load Balancing & Scaling： 可分散負載並隨時調整 POD 的數量

實現的方式有兩種 Replication Controller 與 ReplicaSet，其中 Rplication Controller 被 RepliceSet 取代。
主要差異如下：
* Seletor：ReplicaSet 是 set-based 與 ReplicationController 是 equality-based，簡單來説 set-based就是 tier in (frontend, backend)，equality-based 就是 tier=frontend，前者靈活了不少。
* Update：ReplicationController 只支援 rolling-update 指令更新，而 ReplicaSet 可以透過 Deployment 做出更靈活的更新方式，像是 rollout。

比較表
| Replication Controller | Replica Set |
|---|---|
| The Replication Controller is the original from of replication in Kubernetes | ReplicaSets are a higher-level API that gives the ability to easily run multiple instance of a given pod |
| The Replication Controller uses __equality-based selectors__ to manage the pods.| ReplicaSet Controller user __set-baed selectors__ to manage the pods.|
|The rolling-update command works with Replication Controllers| The rolling-update command won't work with ReplicaSets.|
|Replica Controller is deprecated and replace by ReplicaSet| Deployments are recommended over RplicaSets.|

## 1. Replication Controller



## 2. ReplicaSet

    not ste selector

spec template


replace set

Labael and selector


kubectl creat -f replicate-definetion.yam

kubectl get replicaset

kubectl delete replicast