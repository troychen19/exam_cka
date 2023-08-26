# 網路管理

**重點**  
* 創建及刪除網路策略
* 創建基於標籤的網策略

網路策略其實類似防火牆，允許哪些可訪問，哪些不能訪問，主要有兩種類型
* ingress：用來限制 "進" 的流量
* egress：用來設置 pod 數據是否能出去

## 建立測試環境

1. 建立兩個 pod， pod1、pod2，他們的標籤分別為 run=pod1 與 run=pod2，分別為這兩個 Pod 建立 service，svc1、svc2
   ```bash
    kubectl run pod1 --image=nginx --image-pull-policy=IfNotPresent --labels=run=pod1 --dry-run=client -o yaml > pod1.yaml
    kubectl run pod2 --image=nginx --image-pull-policy=IfNotPresent --labels=run=pod2 --dry-urn=client -o yaml > pod2.yaml
    kubectl get pods --show-labels

    ## 修改 nginx index 內容
    kubectl exec -it pod1 -- sh -c "echo 111 > /usr/share/nginx/html/index.html"
    kubectl exec -it pod2 -- sh -c "echo 222 > /usr/share/nginx/html/index.html"

    ## 測試
    kubeclt get pods -o wide
    curl http://<ip>

    ## 建立 svc
    kubectl expose pod pod1 --name=svc1 --port=80 --type=NodePort
    kubectl expose pod pod2 --name=svc2 --port=80 --type=NodePort

    ## 檢查 svc
    kubectl get svc -o wide
   ```
2. 建立一個網路策略 (netwrok policies)，應用在 svc1 上，然後測試是否還可以訪問 pod1
   ```bash
    ## 檢查是否有網路策略存在
    kubectl get networkpolicies

    ## 創建一個測試 clinet busybox 測試在没建 networkpolicy 前訪問不受限制
    alias k-busybox="kubectl run busybox --rm -it --image=busybox --image-pull-policy=IfNotPresent"
   ```
   * 測試 ingress
     建立 networkpolicies pod1 只允許標籤 xx=xx 的訪問，spec:podSelector 設置策略應用在哪些 pod 上,要保護哪些 pod，spec.imgress.from 設置是允許哪些 client 訪問
     ```yaml
        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
        name: mypolicy
        spec:
        podSelector:
            matchLabels:
            run: pod1
        policyTypes:
        - Ingress
        ingress:
        - from:
            - podSelector:
                matchLabels:
                xx: xx
        ports:
        - protocol: TCP
          port: 80
     ```


