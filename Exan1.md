1. RBAC 授權 (強制記憶，3條命令)
設置配置環境
```bash
kubectl config use-context k8s
```
Context  
為譯管道建一個新的 ClusterRole 並將其綁定到範圍特定的 namespace的特定 ServiceAccount

Task  
創建一個名為 depoymemt-clusterrole 且僅允許創建以下資源類型的新 ClusterRole：  
* Deployment 
* StatefulSet 
* DaemonSet

在現有的 namespace app-team1 中創建一個名為 cicd-token 的新 ServiceAccount  
限於 namespace app-team1，將新的 ClusterRole deployment-clusterrole 綁定到新的 ServiceAccount cicd-token

ANS:
解題技巧
1. 創建 clusterrole (Reference/API AccessControl/Using RBAC Authorization)
2. 建立 serviceaccount
3. 將 cluserrole 與 serviceacount 綁定
```bash
kubectl config use-context k8s

kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployment,daemonsets,statefulsets

kubectl create serviceaccount cicd-token -n app-team1

kubectl create rolebinding cicd-token --serviceaccount-app-team1:cicd-token --clusterrole=deployment-clusterrole -n app-team1

#verify:
kubectl describe rolebinding cicd-token-rolebinding -n app-team1

```
---

2. 統計使用 CPU 最高的 Pod (強制記記憶)
設置配置環境
```bash
kubectl config use-context k8s
```
Task  
通過 pod label name=cpu-utilizer，找到運行時佔用大量 CPU 的 pod，並將佔用 CPU 最高的 pod 寫入文件
/out/KUTR00401/KUTR00401.txt

```bash
kubectl config use-context k8s
kubectl top pods -l name=cpu-utilizer --sort-by="cpu" -A 
echo "<podname>" > /out/KUTR00401/KUR00401.txt
```

---

3. 網路策略 (拷貝 yaml) (注意 yaml 位置)
```bash
kubectl config use-context hk8s
```
Task  
在現有的 namespace my-app 中創建一個名為 allow-port-from-namespace 的新的 NetworkPolicy
確保新的 NetworkPolicy 允許 namespace my-app 中的 Pods 來連接到 namesapce big-corp 的端口 8080
進一步確保新的 NetworkPolicy:
**不允許**對没有在監聴端口 8008 的 pod 訪問
**不允計**不來自 namespace my-app 中的 Pods 訪問

ANS：
```bash
kubectl config use-context hk8s
```
```yaml
aplVersion: netowrking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: my-app
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSecector:
        matchLabels:
          name: big-corp
    ports:
    - protocol: TCP
      port: 8080
```

Verify:
```bash
kubectl get networkpolicy -n my-app
```

---

4. SVC 暴露應用
```bash
kubectl config use-context k8s
```
請重新配置現在的部屬 front-end 以及添加名為 http 的端口規範來公開現有容哭 nginx 的 端口 80/tcp
創建一個名為 front-end-svc 的新服務，以公開容器端口 http
配置此服務，以通過排定的節點上的 NodePort 來公開整個 Pods

```bash
kubectl config use-context k8s

kubectl edit deploy front-end 
##修改 ports 設定， name: http, portocol: TCP, port 為 80
```yaml
    containers:
    - image: nginx
      imagePullPolicy: Always
      name: nginx
      ports:
      - name: http #额额，这里要加一个-横杠的。。。
        protocol: TCP #protocol不写也是可以的，因为默认就是TCP
        containerPort: 80
```
kubect expose pod front-end --port 80 --target=80 --type=NodePort  --name front-end-svc
```
---

5. Ingress 創建 (拷貝 yaml) (注意 yaml 位置)
環境
```bash
kubectl config use-context k8s
```

Task  
如下創建一個新的 nginx Ingress 資源  
名稱： pong  
Namespace: ing-internal
使用服務端口 5678 在路徑 /hello 上公開服務 hello
```
可以使用以下命令檢查服務 hello 的可用性，該命令返回 hello
curl -kL <Internal_ip>/hello
```

ANS:  
kubectl config use-context k8s
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pong
  namespace: ing-internal
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 5678
```

---

6. 擴充 deployment 副本數量 (強制記憶)
環境
```bash
kubectl config use-context k8s
```

Task    
將 deployment 從 loadbalancer 擴展至 5 pods

ANS:  
```base
kubectl config use-context k8s
kubectl scale deploy loadbalance --replicas=5 
```

---

7. 調度 pod 到指定節點  
環境
```bash
kubectl config use-context k8s
```

Task    
按如下要求調度一個 pod
名稱： nginx-kusc00401
Imange: nginx
Node Selecot: disk=ssd

ANS:

1. label node disk=ssd
2. add pod nodeselet disk=ssd

```bash
kubectl label node node01 disk=ssh

```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disk: ssd
```

---

8. 查看可用節點數量
環境
```bash
kubectl config use-context k8s
```

Task:  
檢查有多少 worker node 已淮備就緒 (不包括被打上 Taint:NoSchedule 的節點)
並將數量寫入 /opt/KUSC00402/kusc00402.txt

ANS:  
```bash
kubectl config use-context k8s

kubectl get nodes --show-labels | grep ready | gerp -v NoSchedule

echo <number> > /opt/KUSC00402/kusc00402.txt
```

9. 創建多容器的 pod
環境
```bash
kubectl config use-context k8s
```
Task  
創建一個名為 kucc4 的 pod，在 pod 裏面分為以下每個 image 單獨運行一個 app container (可能會有 1-4個 images)
nginx + redis + memcached

ANS:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kucc4
spec:
  containers:
  - name: nginx
    image: nginx

  - name: redis
    image: redis

  - name: memcached
    image: memcached
```

10. 創建 PV