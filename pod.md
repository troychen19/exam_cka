# POD 說明

# POS 會考題方向 

在 Pod 的題目範圍，
1. 使用指令 kubectl 建立/刪除/修改 POD
2. 使用 yaml 檔建立 pod，需要善用 kubectl --dry-run=cleint 產生 yaml 檔來修改
3. 使用 kubectl 來觀查 pod 的數量，是否有錯誤
4. 如何解讀 kubectl get pods 的訊息，如 READY 的 0/1 數字表示 `pod 中的 contaioner 啟動數/pod 中的 container 總數 

# POS 指令

1. run pod
```shell
kubectl run nginx --image nginxs
```

2. run pod use YAML

| kind | Version |
| --- | --- |
| POD | v1 |
| Service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
      app: myapp
spec:
  containers
    - name: nginx-container
      image: nginx
```