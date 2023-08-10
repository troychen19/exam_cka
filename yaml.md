yaml 文件的寫法是分級的，子級與父級問要縮排 2 個 space，子級的第一個位置可以 "-" 開頭。"-" 需要和父級對齊。
例：
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1 # 標籤設置
  name: pod1  # pod 名稱
  namespace: default # 指定 pod 所在的 namespace，不指定則預設在當前的 namespace
space:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: pod1
    resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Always
status: {}
```

查看 pod 有什麼參數可用 kubectl explain pods
在 containers 的重點說明，下載策略 imagePullPolicy
1. Always：不管本有没有鏡像，都會到網路下載
2. Never：只用本地的鏡像，如本地不存在就報錯
3. IfNotPresent：優先使用本地鏡像，如果本地不存在則去網路下載