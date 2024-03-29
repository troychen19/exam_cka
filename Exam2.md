# KodeKloud Exam2

# 1. Create a new pod called `super-user-pod` with image `busybox:1.28. Allow the pod to be able to set `system_time`.
The container should sleep for 4800 seconds.  

## ANS:
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
* 允許 pod 設定 system_time。

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

# 2. pv 與 pvc 概念的釐清，與 pod 的使用
pvc 自動綁定 pv

## ANS

# 3. deploy rolling upgrade
Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Next upgrade the deployment to version 1.17 using rolling update.

## ANS

```
To create a resource from definition file and to record:
kubectl apply -f deploy.yaml --record

To view the history of deployment nginx-deploy:
kubectl rollout history deploy nginx-deploy

To upgrade the image to next given version:
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record


kubectl rollout history deploy nginx-deploy
```

# 4. 建立使用者
Create a new user called john. Grant him access to the cluster. John should have permission to create, list, get, update and delete pods in the development namespace . The private key exists in the location: /root/CKA/john.key and csr at /root/CKA/john.csr.

Important Note: As of kubernetes 1.19, the CertificateSigningRequest object expects a signerName.

Please refer the documentation to see an example. The documentation tab is available at the top right of terminal.

## ANS
Reference-> API Access Control -> Certificates and Certificate Signing Request
1. create CSR and approve
```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth
  ```
To approve this certificate, run: 
```
kubectl certificate approve john-developer
```
2. create role
```
$ kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development
```
3. create rolebinding
```
$ kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development
```



# 5. 建立 pod、service 後，使用 nslookup 查詢兩個服務的 DNS

## AMS
Concepts -> Services Load Balancing and Networking

Service: <svc name>-n<>
Pod: IP address is 172.17.0.3, 172-17-0-3.default.pod.cluster.local