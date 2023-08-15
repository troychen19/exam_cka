# 一、 emptyDir
**重點** 創建 emptyDir 類型的 volume，並掛載
使用 emptyDir 就如同 docker 容器的命令 docker run -v /xx，意思就是在物理機隨機產生一個目錄，然把這個目錄掛載到容器的 /xx 目錄。當 pod 刪除時 emptyDir 目錄也會一併刪除。
1. 建立 volume 目錄 `mkdir volume`
2. 建立 yaml 檔
```bash
  kubectl run demo --dry-run=client --image=busybox -o yaml > emp.yaml
```
3. 修改 yaml 
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: demo
  name: demo
spec:
  volumes:
  # 第一個 volume
  - name: volume1
    emptyDir: {}
  # 第二個 volume
  - name: volume2
    emptyDir: {}
  containers:
  - name: demo1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    # volume1 掛載到 /v1
    volumeMounts:
    - mountPath: /v1
      name: volume1

  - name: demo2
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    # volume1 掛載到 /v2
    volumeMounts:
    - mountPath: /v2
      name: volume1

```
使用 `kubectl describe` 檢查 pod 可看到 /v1, /v2 都掛載到了 /volume1


# 二、 hostPath
# 三、 使用 NFS
# 四、 配置持久性儲存
# 五、 配置動態 volume