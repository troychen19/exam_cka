# 密碼管理
**重點**
1. 創建及刪除 secret
2. 在 POD 使用變量的方式引用 secret
3. 在 POD 以 volume 的方式引用 secret
4. 創建及刪除 configmap
5. 在 POD 使用變量的方式引用 secret
6. 在 POD 以 volume 方式引用 secret

# 一、 secret

**重點** 創建 secret 後，用變量與卷的方式使用

secret 主要用來存放密碼，secret 用鍵值的方式存儲。 secret 有三種類型
* Opaque： base64 編碼格式的 secret，用來存放密碼、密鑰
* kubernetes.io/dockerconfigjson：用來儲放私有的 docker registry 認證
* kubernetes.ip/service-account-token：用於被 serviceaccount 引用
1. 查看 secret
   ```bash
     kubectl get secrets -n default
   ```
2. 使用命令建立 secret
   ```bash
    kubectl create secret generic mysecret1 --from-literal=user1-pw=abc1234
   ```
3. 檢查建立的 secret
   ```bash
     kubectl describe secrets

     # 查看鍵值
     kubectl get secret mysecret1 -o yaml
   ```
4. 將文件創建為 secret