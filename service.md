# 服務管理

---
**重點**  
1. 創建/刪除 service
2. 了解 service 是透過 labels 定位 pod
3. 通過 NodPort 發佈服務
4. 創建 ingress 發佈服務
---

service 用來做為 pod 與外部的轉接口，可以把 service 理解為一個負載平衡器，所有的 request 透過 service 轉發給後端的 pod。service 之所以能把 request 轉發給後端的 pod，是由 kube-proxy 來實現。service 是透過標纖 (label) 來定位 pod，deployment 創建出來的 pod 都具有相同 label。所以如果某個 pod 掛了，deployment 會生成一個同標籤的 pod，此時 service 能立即定位找到新的 pod

## 服務的基本管理  

1. 建立一個 deployment, 名稱 web, image: nginx 
   ```bash
    kubectl create deploy web --image=nginx --dry-run=client -o yaml > web.yaml
   ```
2. 建立 svc  
expose 的目標可以是 rc (replica) 、 svc (service)、 pod、 deployment，port 是指服務的端口， target-port 則是 pod 運行的端口，需要依運行的 pod 所使用的 port
   ```bash
    kubectl expose deployment web --name web-svc --port 80 --target-port 80
   ```
3. 查看建立的 pod  
可看到 svc 是透過 selector 來定位 pod
   ```bash
     kubect get svc -o wide
   ```
4. 擴展副本數到 3  
   ```bash
    kubectl scale deploy web --replicas=3
   ```
5. 查看 svc 的 endpoint 也加入了新的 pod ip
6. 刪除 svc
   ```bash
    kubectl delete svc web
   ```
7. 使用 yaml 的方式建立 svc
   ```bash
    kubectl expose deploy web --name svc1 --port=80 --target-port=80 --dry-run=client -o yaml > svc1.yaml
   ```

## 服務發現  
**重點** 通過 clusterIP、變量、DNS 發現 svc

當建立 wordpress 服務時，需要使用 mysql，在部署完 mysql 並建立 svc 後，可透過 clusertIP、環境變量與 dns
* 環境變量，當同一個 namespace 已存在 A svc，那麼 pod B 部署的時候，會自動取得 A 服務的相關變量
   ```
    A服務名稱_SERVICE_HOST
    A服務名稱_SERVICE_PORT
   ```
* 在 kubernetes 安裝完畢後，在 kube-system 的 namespace 會有 CoreDNS，作為內部 pod 的 DNS 服務

## 服務發佈
**重點** 透過 NodPort 或 ingress 來發佈服務

```bash
 kubectl expose svc web-svc --name=myweb --port=80 --type=NodePort
```