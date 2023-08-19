# 探針

---

**重點**  
1. 為 pod 增加 liveness 探針
2. 為 pod 增加 readiness 探針

---

對 deployment 來說只管 pod running 就好，而不管 Pod 內容是否運作正常。這時候就要靠設置 probe 來查看 pod 內部的元件是否正常。

## liveness probe  
**重點**： 使用 liveness command 與 liveness httpGet 的方式探測  
1. liveness command 
使用 dry-run 產生 yaml 後修改如下：
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: liveness
    name: liveness-exec
    spec:
    containers:
    - image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 1000
        livenessProbe:
        exec:
            command:
            - cat
            - /tmp/healthy
        initialDelaySeconds: 5
        periodSeconds: 5
    dnsPolicy: ClusterFirst
    restartPolicy: Always
   ```
livenessProb 的檢查方式，啟動時 deloy 5 秒，之後每五秒檢查一次 healthy 是否存在，在 image 啟動時執行 script 則是產生 healthyf，接著 30 秒後刪除，然後等等 1000 秒，因此探針在 30 秒後發現檔案不在就重啟
   ```bash
     kubectl describ pod liveness-exec
     Events:
     Type     Reason     Age                     From               Message
     ----     ------     ----                    ----               -------
     Normal   Scheduled  9m30s                   default-scheduler  Successfully assigned default/liveness-exec to node02
     Normal   Killing    6m15s (x3 over 8m45s)   kubelet            Container busybox failed liveness probe, will be restarted
     Normal   Pulled     5m45s (x4 over 9m30s)   kubelet            Container image "busybox" already present on machine
     Normal   Started    5m45s (x4 over 9m30s)   kubelet            Started container busybox
     Warning  Unhealthy  5m10s (x10 over 8m55s)  kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
     Normal   Created    4m30s (x5 over 9m30s)   kubelet            Created container busybox
   ```
除了 initialDelaySeconds 與 periodSeconds 外，還有兩個重要參數
   * successThreshold：探測失敗後，最少連續幾次成功才被設定成功，預設 1
   * failureThreshold：探測失敗後 kubernetes 重啟次數，預設值 3
2. liveness httpGet  
httpGet 的方式，指的是 HTTP 協議的數據包能否通過指定的端口訪問到指定的文件，如果可以代表正常，如果訪問不到，表示異常
修改 yaml 檔，image 使用 nginx 並檢查 index.html 是否能訪問
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: liveness
    name: liveness-http
    spec:
    containers:
    - image: nginx
        imagePullPolicy: IfNotPresent
        name: nginx
        livenessProbe:
          failureThreshold: 3
          httpGet:
              path: /index.html
              port: 80
              scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
    dnsPolicy: ClusterFirst
    restartPolicy: Always
   ```

3. liveness tcpSocket  
tcpSocket 的探測方式是指，能否和指定的端口建立 tcp 三次握手，如果能則探測通過，下面的方式是探測 80 port 是否可通
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
    creationTimestamp: null
    labels:
        run: liveness
    name: liveness-scoket
    spec:
    containers:
    - image: nginx
        imagePullPolicy: IfNotPresent
        name: nginx
        livenessProbe:
          failureThreshold: 3
          tcpSocket:
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          successThreshold: 1
    dnsPolicy: ClusterFirst
    restartPolicy: Always
   ```

---

## readiness probe  
readiness 與 liveness 的探測方式一樣，但是發現問題後處理方式不同。
* liveness 收到異常後，透過重啟來解決
* readiness 收到異常後不重啟，但是收到 svc 的 request 後，不再轉發到有問題的 pod

1. 建立 pod 使用 readiness 的 yaml  
   ```bash
    kubectl create deploy readiness-nginx --imgae=nginx --replicas 3 --dry-run=client -o yaml > readiness-nginx.yaml
   ```
   修改檔案，加入 readiness 設定
   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    creationTimestamp: null
    labels:
        app: readiness-nginx
    name: readiness-nginx
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: readiness-nginx
    template:
        metadata:
        creationTimestamp: null
        labels:
            app: readiness-nginx
        spec:
        containers:
        - image: nginx
            imagePullPolicy: IfNotPresent
            name: nginx
            lifecycle:
              postStart:
                exec:
                command: ['/bin/sh','-c','touch /tmp/healthy']
            readinessProbe:
            exec:
                command:
                - cat
                - /tmp/healthy
   ```
2. 建立名字為 readiness-nginx 的 svc  
   ```bash
    kubectl expose deploy readiness-nginx --port=80 --target-port 80
   ```
3. 將 3 個 pod 的 index.html 內容改為 111,222,333  
分別在 3 個 node 執行以下指令
  ```bash
   kubectl exec -it readingss-nginx-xxx -- bash
   # 進入 pods 後執行以下指令
   each '111' > /usr/share/nginx/html/index.html
  ```
4. 透過 svc 訪問 pod  
可看到回應的內容為 111, 222 或 333
5. 刪除 pod3 的 /temp/health，讓 pod3 探測失敗
   ```bash
    kubectl exec readingess-nginx-xxx -- rm /tmp/healthy
   ```
6. 再次透過 svc 訪問，不會出現 pod3