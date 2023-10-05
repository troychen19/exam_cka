# Kubernetes Service Account
service account 是由 kubernetes 管理的帳戶類型，在管理上可說是方便，但在剛接觸時，不是很容易理解應用情境。

## 帳號類型
kubernetes 的帳號類型有兩種：
1. 使用者帳戶 (Normal Users)
任何人想要連接並存取 kubernetes cluster，都要先建立一個 **使用者帳戶** ，並將憑證提供給用戶端 (kubectl)，以便通過 kubernetes 的 API Server 認證 (Authentication)
2. 服務帳戶 (Service Accounts)
仕何執行在 Pod 的 container 想要存取 Kubernetes 的 `API 伺服器` (kube-apiserver)，就需要有一個  **服務帳戶** 綁定。然後通過 Kubernets API 伺服器的身份認證

## 體驗命名空間預設的服務帳戶 (Service Account)
當建立 namespace 後，會自動建立 `default` 的 service account
1. 建立 namespace - dev
2. 檢視 namespace - `dev`  下的 `service account`
3. 檢視 dev 下的 default 內容
```bash
 kubectl get serviceaccount default -n dev -o yaml
```
```yml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2023-10-03T06:58:03Z"
  name: default
  namespace: dev
  resourceVersion: "104197"
  uid: 8c9c2640-0872-4a64-a7e1-0a62c922c938
```

## 體驗 Pod 如何使用服務帳戶 (Service Account)