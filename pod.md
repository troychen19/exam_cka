# POD 說明

POD 為 k8s 用來封裝管理 container 的單元，一個 POD 可含有多個 container。 

在 YAML 檔設定，不同 kind 可以用的 apiVersion 也不同，整理如下表
| kind | apiVersion |
| --- | --- |
| POD | v1 |
| Service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

# POS 會考題方向 

在 Pod 的題目範圍，
1. 使用指令 kubectl 建立/刪除/修改 POD
2. 使用 yaml 檔建立 pod，需要善用 kubectl --dry-run=cleint 產生 yaml 檔來修改
3. 使用 kubectl 來觀查 pod 的數量，是否有錯誤
4. 如何解讀 kubectl get pods 的訊息，如 READY 的 0/1 數字表示 `pod 中的 contaioner 啟動數/pod 中的 container 總數 

# POD 題目

1. 使用 image nginx 建立名為 nginx 的 POD
```shell
kubectl run nginx --image nginx
```

2. 使用 YAML 建立 iimage 為 nginx123 的 POD
    * 使用 dry-run 參數產出 YAML 檔
      ```bash
        kubectl run nginx --image nginx123 --dry-run=client -o yaml > nginx.yaml
      ```
      執行後產出的 YAML 如下
      <details>
          <summary>nginx.yaml</summary>
          apiVersion: v1
          kind: Pod
          metadata:
            creationTimestamp: null
            labels:
              run: nginx
            name: nginx
          spec:
            containers:
            - image: nginx123
              name: nginx
              resources: {}
            dnsPolicy: ClusterFirst
            restartPolicy: Always
          status: {}
      </details>
    * 執行建立
      ```bash
        kubect apply -f nginx.yaml
      ```

3. 檢查 POD 是否正常啟動，錯誤的原因是什麼

    使用 get pods 檢查執行錯誤是 ImagePullBackOff
    ```bash
      kubectl get pods

      NAME    READY   STATUS             RESTARTS   AGE
      nginx   0/1     ImagePullBackOff   0          48s

    ```

    使用 describe pod 查看詳細原因是 image 名稱錯誤
    ```bash
      kubectl describe pod nginx

      ...

      
      Events:
        Type     Reason     Age                   From               Message
        ----     ------     ----                  ----               -------
        Normal   Scheduled  4m9s                  default-scheduler  Successfully assigned default/nginx to node01
        Normal   Pulling    2m31s (x4 over 4m9s)  kubelet            Pulling image "nginx123"
        Warning  Failed     2m29s (x4 over 4m6s)  kubelet            Failed to pull image "nginx123": rpc error: code = Unknown ...
        Warning  Failed     2m29s (x4 over 4m6s)  kubelet            Error: ErrImagePull
        Warning  Failed     2m13s (x6 over 4m6s)  kubelet            Error: ImagePullBackOff
        Normal   BackOff    2m (x7 over 4m6s)     kubelet            Back-off pulling image "nginx123"

    ```

4. 修正這個錯誤
    修改 nginx 的 yaml 檔，將 image 的值 nginx123 改為 nginx 
    ```bash
      kubectl edit pod nginx
    ```

