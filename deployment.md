| 目錄 |
| --- |
| 1. [創建 deployment 與檢測](#創建-deployment-與檢測) |
| 2. [修改副本數](#修改副本數) |
| 3. [水平自動更新 HPA](#水平自動更新-HPA) |
| 4. [deployment 鏡像的升級與回滾](#deployment-鏡像的升級與回滾)|
| 5. [滾動升級](#滾動升級)|

# deployment  
---
**重點**  
1. 創建/刪除 deployment
2. 增加/減少 pod 副本 (replicas) 數
3. 更新及回滾容器所使用的鏡像 (image)
4. 在線修改 deployment 設置
---

deployment (簡稱 deploy) 是一個控制器，只要告訴 deployment 需要的 pod 數，那麼如果有 pod 掛了，會再生成新的 Pod

## 創建 deployment 與檢測

### 1.1 建立 deployment

1. 用命令的方式生成，語法如下  
   ```bash
    kubectl create deployment mydeploy1 --image=busybox --dry-run=client -o yaml > d1.yaml
   ```
deployment 使用 template 定義 image，metadata、selector 與 template 的名稱必需相同。修改產出的 yaml，將 replicas 改為 3
   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: mydeploy1
      name: mydeploy1
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mydeploy1
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            app: mydeploy1
        spec:
          containers:
          - image: nginx
            name: nginx
            resources: {}
    status: {}
   ```
2. 查看 deployment
```bash
  kubectl get deployment -n default
```

### 1.2 deployment 健狀性測試

測試將其中一個主機關閉，一段時間後關閉主機的 pod 會轉移到正常主機。

---

## 修改副本數

1. 使用命令修改
   ```bash
     kubectl scale deployment mydeploy1 --replicas=4
   ```
2. 直接編輯 deployment  
使用 edit 指令編輯 replicas
   ```bash
     kubectl edit deploy mydeploy1
   ```
3. 修改 yaml 文件
修改 yaml 檔後執行 apply

---

## 水平自動更新 HPA

水平自動更新 (Horizontal Pod Autoscalers, HPA)，通過檢測 pod 的 CPU 負載通知 deployment，自動更新 pod 的數量。
 1. 直接使用命令創建 HPA  
語法說明： 意思是此 deployment 最少運行 M 個 pod，確保每個 CPU 使用率最大不超過 x%，否則擴展副數到 N
    ```bash
      kubectl autoscale deployment 名字 --min=M --max=N --cpu-percent=x
    ```
例：
   ```bash
    kubectl autoscale deployment mydeploy1 --min=2 --max=5
   ```
檢視/刪除 hpa
   ```bash
    kubectl get hpa
    kubectl delete hpa mydeploy1
   ```
`--cpu-percent` 要生效，必須要啟用 deployment 的資源請求。修改 deployment
  ```yaml
   ...
   template:
     metadata:
       creationTimestamp: null
         labels:
           app: mydeploy1
       spec:
         containers:
         - image: nginx
           name: nginx
           resources:
             requests: 
               cpu: 400m

  ```
再次建立 hpa
   ```bash
    kubectl autosacle deploy mydeploy1 --min=1 --max=5 --cpu-percent=80
   ```

---

## deployment 鏡像的升級與回滾

1. image 升級  
有三種方式
   * kubectl edit deploy
   * 修改 yaml 後再次執行 kubectl apply -f yaml
   * 命令列修改 `kubectl set image deployment/名字`
例：
將 deployment 的 ngixn 換為指版本
   ```bash
     kubectl set image deployment/mydeploy1 nginx=nginx:1.7.9 --record
   ```
查看鏡像變化過程
   ```bash
     kubectl rollout history deployment mydeploy1
   ```
當升級版本有問題需要回滾，--to-revision 為指定要回復指定那次變更
  ```bash
   kubectl rollout undo deployment mydeploy1 --to-revision=1
  ```

---

## 滾動升級  
滾動更新就是，不一次性的將 pod 的鏡像更新，而是分批次更新。每次更新的次數可以透過參數設定
1. maxSuge： 用來設定一次最多創建幾個 pod，可以是百分比也可以是數字
2. maxUnavailable：指定最多刪除多少 pod，可以是百分比也可以是數字
執行 `kubectl edit deploy mydeploy1` 可以看到設定
```yaml
...
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: mydeploy1
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
...
```
