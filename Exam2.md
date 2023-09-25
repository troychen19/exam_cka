# KodeKloud Exam2

1. Create a new pod called `super-user-pod` with image `busybox:1.28. Allow the pod to be able to set `system_time`.
The container should sleep for 4800 seconds.  

# ANS:
這題考的是 Pod的安全策略，雖然Pod是受到 kubernetes 經過檢查確認合法才得以部署的，但是由於這些服務都會直接面向User，若這些容器內本身的權限過高且遭受到攻擊，就會衍生出其他的安全性問題。SecurityContext就是用來解決這類問題的，它定義了Pod或容器的特權和訪問控制設置。SecurityContext包括：
* Discretionary Access Control: 訪問目標（如檔案）的權限基於User ID（UID）和 Group ID（GID）。
* Security Enhanced Linux (SELinux): 為目標分配安全標籤
* Running as privileged or unprivileged: 以特權或非特權運行
* Linux Capabilities: 為某些process提供特權，但不是root的所有特權
* AppArmor: 使用程式配置文件來限制個別程式的功能。
* Seccomp: 過濾及篩選process的system call
* AllowPrivilegeEscalation: 控制process是否可以比其parent process獲得更多的特權。
* readOnlyRootFilesystem: 將容器的root file system mount 為 Read-Only。
回到這題。這題有兩個重點：
* 此container創建後要sleep 4800秒
* 允許pod設定system_time。

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: super-user-pod
  name: super-user-pod
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox:1.28
    name: super-user-pod
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

2. pv 與 pvc 概念的釐清，與 pod 的使用
pvc 自動綁定 pv

3. deploy rolling upgrade
```
kubectl apply -f deploy.yaml --record
kubectl rollout history deploy nginx-deploy
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
kubectl rollout history deploy nginx-deploy
```
4. 建立使用者

5. 建立 pod、service 後，使用 nslookup 查詢兩個服務

6. 