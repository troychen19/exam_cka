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
2. 使用命令建立 secret, 一定要加參數 gereric 表示使用通用的方式建立
   ```bash
    kubectl create secret generic mysecret1 --from-literal=user1=abc1234
   ```
3. 檢查建立的 secret
   ```bash
    # 查看 mysecret1 的內容
    kubectl describe secrets mysecret1

    # 查看 mysecret1 鍵值
    kubectl get secret mysecret1 -o yaml
   ```
4. 將文件內容寫入 secret  
以下範例將 hosts 內容寫入 secret, 一定要加參數 gereric 表示使用通用的方式建立
   ```bash
    kubectl create secret generic mysecret2 --from-file=/etc/hosts
   ```
   使用 base64 檢視
   ```bash
    kubectl get secret mysecret2 -o jsonpath={.data.hosts} | base64 -d
   ```
5. 建立變量文件並寫入 secret  
建立變文件，文件內容編寫格式為 `變量1=值1`
   ```
    # 建立檔案 var.txt
    key1=value1
    key2=value2
   ```
   將 var.txt 寫入 secret, 一定要加參數 gereric 表示使用通用的方式建立
   ```bash
    kubectl create secret generic mysecret3 --from-env-file=var.txt
   ```
   檢查內容，可發現檔案內容的 key1, key2 變成 secret 的 key 值
   ```bash
    kubect get secret mysecret3 -o jsonpath={.data.key1} | base64 -d
   ```
6. 使用 yaml 文件建立 secret  
value 值必需是 base64，需注意 type 可加可不加，Opaque 表示不透明
   ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysecret4
      namespace: mysecret
    type: Opaque
    data:
      key1: dmFsdWUx
      key2: dmFsdWUy
   ```
7. 使用 secret  
方法一： **使用卷的方式**  
這種方式是在 pod 的 yaml 建立 secret 的卷，然後掛載到某個指定目錄
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sec_volum_demo
      labels:
        run: nginx
    spec:
      volumes:
      - name: mysecret
        secret:
          secretName: mysecret1
     containers:
     - name: pod1
       image: nginx
       imagePullPolicy: IfNotPresent
       volumeMounts:
       - name: mysecret
         mountPath: /mysecret
   ```
檢查 /mysecret 是否出現 key 值
   ```bash
     kubectl exec sec-vol-demo -- ls /mysecret
   ```
使用此方式可以將 nginx 配置寫在 secret 就不用重新編譯 image 了
方法二： **使用變量的方式**  
在 pod 中想用變量的話，格式為
   ```
    env:
      - name: 變量名
        value: 值
   ```
如果要使用 sercet 則將 value 改為 valueFrom
範例： 以下的 mysql root password 的值使用 mysecret1 的 user1
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: var-sec-demo
    spec:
      containers:
      - name: pod2
        image:  mysql:latest
        imagePullPolicy: IfNotPresent
      env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret1
              key: user1
   ```
方法三： **指定使用 secret**
```yaml
 apiVersion: v1
 kind: Pod
 metadata:
   name: simple-webapp-color
 spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    envFrom:
    - secretRef:
        name: mysecret1
```
二、 configmap  
configmap (簡稱 cm) 的作用和 secret 一樣，作為儲放密碼或 pod 的文件。
2.1 創建 configmap  
1. 使用命令的方式
   ```bash
    kubectl create cm my1 --from-literal=xx=tom --from-literal=yy=redhat
   ```
2. 將文件寫入 configmap
   ```bash
     kubectl create cm my2 --from-file=/etc/hosts
   ```
3. 檢視 configmap
   ```bash
     kubectl describe cm my1

     kubectl get cm my1 -o yaml
   ```
可用 dry-run 來建立基本的 temp，再修改值
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cfmap-demo
data:
  KEY1: value1
  USER: myname
```

2.2 使用 configmap  
方法一：卷的方式  
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-cm1-demo
    spec:
      volumes:
      - name: xx
        configMap:
        name: my1
      containers:
      - name: nginx1
        image: nginx
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: xx
          mountPath: /mycm1   
   ```
方法二：變量的方式  
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: var-cm1-demo
    spec:
      containers:
      - name: pod2
        image:  mysql:latest
        imagePullPolicy: IfNotPresent
      env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            configMapRef:
              name: mycm1
              key: yy
   ```
方法三：指定使用的 configmap
   ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: simple-webapp-color
    spec:
    containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
      - containerPort: 8080
      envFrom:
      - configMapRef:
          name: cfmap-demo
   ```