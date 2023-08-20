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


## 服務發現  


## 服務發佈
