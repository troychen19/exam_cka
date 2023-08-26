# 安全管理  
在 k8s 上，只要登入 master 就可以用 kubectl 的各種命令進行操作，是因為通過 kubeconfig 來驗證。
這裏的 kubeconfig 並不是文件而是憑證，存在 home 目錄的 .kube/config 的目錄下，在第一次初始化後從 /etc/kubernetes/admin.conf 複製過來
只要將這個文件複製到任一台 node，該 node 就可操作 kubectl

---

**安裝 kubectx/kubens**  
```bash
  git clone https://github.com/ahmetb/kubectx /opt/kubectx
  ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
  ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

---

## 一、 建立 kubeconfig 文件  
1. 建立憑證
   ```bash
    openssl genrsa -out john.key 2048
    openssl req -new -key john.key -out john.csr -subj "/CN=john/O=cka2020"
    cat john.csr | base64 | tr -d "\n"
   ```
2. 部署憑證並授權使用者
   建立憑證請求文件 yaml
   ```yaml
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
    name: john
    spec:
    signerName: kubernetes.io/kube-apiserver-client
    request: <將剛才的 base64 加密文字貼上>
    expirationSeconds: 86400 # one day
    usages:
    - client auth
   ```
   檢查憑證並核準憑證
   ```bash
    kubectl get csr
    kubectl certificate approve john
   ```
   導出 crt
   ```bash
    kubectl get csr/john -o jsonpath='{.status.certificate}' | base64 -d > john.crt
   ```
   給用戶授權，給 john 一個集群角色 (clusterrole)，以下是將 admin 權限授權給 john
   ```bash
    kubectl create clusterrolebinding test1 --clusterrole=cluster-admin --user=john
   ```
3. 建立 kubeconfig 文件
   建立 kubeconfig 模版文件 kc1
   ```yaml
    apiVersion: v1
    kind: Config
    preferences: {}
    clusters:
    - cluster:
      name: cluster1
    users:
    - name: john
    contexts:
    - context:
      name: context1
      namespace: default
    current-context: "context1"
   ```
   copy ca.crt
   ```bash
    cp /etc/kubernetes/pki/ca.crt .
   ```
   設置集群字段，embed-certs=true 的意思是將此 kubeconfig 寫到 kc1
   ```bash
    kubectl config --kubeconfig=kc1 set-cluster cluster1 --server=https://192.168.153.101:6443 --certificate-authory=ca.crt --embed-certs=true
   ```
   設置用戶字段
   ```bash
    kubectl config --kubeconfig=kc1 set-credentials john --client-certificate=john.crt --client-key=john.key --embed-certs=true
   ```
   設置上下文字段，上下文是把集群和用戶關聯在一起
   ```bash
    kubectl config --kubeconfig=kc1 set-context context1 --cluster=cluster1 --namespace=default --user=john
   ```
4. 驗證  
檢查 johne 是否有 list 的權限
   ```bash
    kubectl auth can-i list pods --as john
    kubectl auth can-i list pods -n kube-system --as john
   ```
將 kc1 複製到 node1 執行
   ```bash
    kubectl --kubeconfig=kc1 get nodes
   ```
將 clusterrolebinds 解除
   ```bash
    kubectl delete clusterrolebings test1
   ```

## 二、 kubernetes 上的授權  
授權一般是基於 RBAC (Role Based Access Control，基於角色的訪問控制)，就是不把權限授權給用戶，而是把幾個權限放在一個角色，再把角色授權給用戶。
apiVersion 中的類型說明：
這些 apiVersion 是以 "父級/子級" 的格式來寫，在定義 role 的 yaml 的 apiGroup 字段，對應父級即可，如果没有父級，如 pod，則在 apiGroup 寫 ""。因為 api、service 没有父級，那麼決定是 pod 還是 service 由 resource 決定
| kind | apiVersion |
| --- | --- |
| pod | v1 |
| service | v1 |
| replicaSet | apps/v1 |
| daemonset | apps/v1 |
| deployment | apps/v1 |
| job | batch/v1 |
| cronjob | batch/v1 |
| role | rbac.authorization.k8s.io/v1 |
| RoleBinding | rbac.authorization.k8s.io/v1 |

1. role 與 rolebinding  
   建立一個對 pods 有檢視權限的 role
   ```bash
     kubectl create role pod-reader --verb=get,watch,list --resource=pods --dry-run=client -o yaml > pod-reader.yaml
   ```
   ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      creationTimestamp: null
      name: pod-reader
    rules:
    - apiGroups:
      - ""
      resources:
      - pods
      verbs:
      - get
      - watch
      - list 
   ```
將角色授權給用戶，由 rolebinding 來完成。這裏是將色角授權給 john
   ```bash
    kubectl create rolebinding rbind1 --role=pod-reader --user=john --dry-run=client -o yaml > rbind1.yaml
   ```
在 node01 上測試，因為 namespace 在 default 建立的，因此只能看 default 的 pod
   ```bash
    kubectl --kubeconfig=kc1 get nods -n ns3
   ```
如果要在全區使用，必須建立 clusterrole 與 clusterrolebinding
   ```bash
    kubectl create clusterrole pod-reader --verb=get,watch,list --resource=deploy --dry-run=client -o yaml > clusterrole1.yaml

    kubectl create clusterrolebinding cbind1 --clusterrole=pod-reader --user=john --dry-run=client -o yaml > cbind1.yaml
   ```

service account
   ```bash
    kubectl create sa app1

    ## run pod use service account
    kubectl set app1 run nginx --image=nginx --image-pull-policy=IfNotPresent 
   ```

## 三、 安裝 dashboard  
## 四、 資源限制  