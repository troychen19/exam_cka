# 1. RBAC 授權 (強制記憶，3條命令)
設置配置環境
```bash
kubectl config use-context k8s
```
Context：  
為譯管道建一個新的 ClusterRole 並將其綁定到範圍特定的 namespace的特定 ServiceAccount

Task：  
創建一個名為 deploymemt-clusterrole 且僅允許創建以下資源類型的新 ClusterRole：  
* Deployment 
* StatefulSet 
* DaemonSet

在現有的 namespace app-team1 中創建一個名為 cicd-token 的新 ServiceAccount，並只能用在 namespace app-team1，
將新的 ClusterRole deployment-clusterrole 綁定到新的 ServiceAccount cicd-token

## ANS:  
解題技巧  
要避免打錯字，可用 dry-run 產 yaml 檔後檢查
1. 創建 clusterrole (Reference/API AccessControl/Using RBAC Authorization)
2. 建立 serviceaccount
3. 將 cluserrole 與 serviceacount 綁定
```bash
kubectl config use-context k8s

kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployment,daemonsets,statefulsets

kubectl create serviceaccount cicd-token -n app-team1

kubectl create rolebinding cicd-token --serviceaccount-app-team1:cicd-token --clusterrole=deployment-clusterrole

#verify:
kubectl describe rolebinding cicd-token-rolebinding 
```

---

# 2. 統計使用 CPU 最高的 Pod (強制記記憶)
設置配置環境
```bash
kubectl config use-context k8s
```
Task  
通過 pod label name=cpu-utilizer，找到運行時佔用大量 CPU 的 pod，並將佔用 CPU 最高的 pod 寫入文件
/out/KUTR00401/KUTR00401.txt

kube
## ANS:
```bash
kubectl config use-context k8s
kubectl top pods -l name=cpu-utilizer --sort-by="cpu" -A 
echo "<podname>" > /out/KUTR00401/KUR00401.txt
```

---

# 3. 網路策略 (拷貝 yaml) (注意 yaml 位置)
```bash
kubectl config use-context hk8s
```
Task  
在現有的 namespace my-app 中創建一個名為 allow-port-from-namespace 的新的 NetworkPolicy
確保新的 NetworkPolicy 允許 namespace big-corp 中的 Pods 來連接到 namesapce my-app的端口 8080
進一步確保新的 NetworkPolicy:
**不允許**對没有在監聴端口 8008 的 pod 訪問
**不允計**非來自 namespace big-corp 中的 Pods 訪問

## ANS：
解題技巧： yaml 位置 Concepts -> Services, Load Balancing, and Networking -> Network Policies
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

# 4. SVC 暴露應用
```bash
kubectl config use-context k8s
```
請重新配置現有的 deployment front-end 以及添加名為 http 的端口規範來公開現有容器 nginx 的 端口 80/tcp
創建一個名為 front-end-svc 的新服務，以公開容器端口 http
配置此服務，以通過排定的節點上的 NodePort 來公開整個 Pods

## ANS
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

```bash
kubect expose pod front-end --port 80 --target=80 --type=NodePort  --name front-end-svc
```
---

# 5. Ingress 創建 (拷貝 yaml) (注意 yaml 位置)
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

## ANS:  
kubectl config use-context k8s
kbuectl get svc -n ing-internal
kubectl create ingress pong --rule=/hello=sample-nginx:5678 -n ing-internal --dry-run=client -o yaml > ping-ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pong
  namespace: ing-internal
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
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

# 6. 擴充 deployment 副本數量 (強制記憶)
環境
```bash
kubectl config use-context k8s
```

Task    
將 deployment - loadbalancer 擴展至 5 pods

## ANS:  
```base
kubectl config use-context k8s
kubectl scale deploy loadbalance --replicas=5 
```

---

# 7. 調度 pod 到指定節點  
環境
```bash
kubectl config use-context k8s
```

Task    
按如下要求調度一個 pod
名稱： nginx-kusc00401
Imange: nginx
Node Selecot: disk=ssd

## ANS:

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

# 8. 查看可用節點數量
環境
```bash
kubectl config use-context k8s
```

Task:  
檢查有多少 worker node 已淮備就緒 (不包括被打上 Taint:NoSchedule 的節點)
並將數量寫入 /opt/KUSC00402/kusc00402.txt

## ANS:  
```bash
kubectl config use-context k8s

kubectl get nodes | grep Ready
kubectl describe nodes | grep Taint

echo <number> > /opt/KUSC00402/kusc00402.txt
```

# 9. 創建多容器的 pod
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

---

# 10. 創建 PV (yaml 位置)
環境
```
kubectl config use-context hk8s
```

Task:  
創建一個名為 app-data 的 persistent volume 容量為 2Gi，
訪問模式為 ReadWriteOne，volume 類型為 hostPath，位於 /svc/app-data

## ANS

Concepts/Stroage/Volume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-data
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /svc/app-data
    type: Directory

```

---

11. 創建 PVC (注意 yaml 位置)
設置環境： 
``` 
kubectl config use-context ok8s
```

Task  
創建一個新的 PersistentVolumeClaim
名稱： pv-volume  
Class: csi-hostpath-sc  
容量： 10Mi  

創建一個新的 Pod，此 Pod 將作為 volume 掛載到 PersistentVolumeClaim  
名稱：web-server  
Iamge: nginx  
掛載路徑： /usr/share/nginx/html
配置新的 Pod，以對 volume 具有 ReadWriteOnce 的權限

最後，使用 kubectl edit 或 kubectl patch 將 PersistentVolumeClaim 的容量擴展為 70Mi，並記錄此更改

## ANS

參考文件： /concepts/storage/persistent volumes
1. Create PersistentVolumeClaim
2. Create Pod use PerstentVolumeClaim
3. kubectl edit pv-volume --record

---

# 12 獲取 Pod 錯誤日誌
```
設置環境：  
kubectl config use-context k8s
```
Task  
監控 pod bar 的日誌：
提取與錯誤 file-not-found 相對應的日誌行
將這些日誌寫入 /out/KUTR00/bar

## ANS

kubectl logs bar | grep ERROR
write to file
echo "<error message>" > /out/KUTR00/bar

---

# 13 使用 sidecar 代理容器日誌 (兩個 pod 共享 pv)
```
設置環境：  
kubectl config use-context k8s
```

Context  
將一個現有的 Pod 集成到 kubernetes 的內置日誌記錄體系結構中 (例 kubectl logs)
添加 streaming sidecar 容器是實現此要求的一種好方法

Task  
使用 busybox Image 來將名為 sidecar 的 sidecar 容器添加到現有的 Pod legacy-app 中，
新的 sidecar 容器必須運行以下命令：
```
/bin/sh -c tail -n+1 -f /var/log/legacy-app.log
```
使用安裝在 /var/log 的 volume，使日誌文件 legacy-app.log 可用於 sidecar 容器  
```
除了添加所需的 volume mount 以外，請勿更改現有容器的規格
```
## ANS  
Concepts/Cluster Administration/Logging Architecture

kubectl get pod big-core-app -o yaml > big-core-app.yaml
kubectl delete big-core-app
## edit big-core-app.yaml
kubectl apply -f big-core-app.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: big-corp-app
spec:
  containers:
  - name: big-corp-app
    image: busybox:1.28
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/11-factor-app.log;
        i=$((i+1));
        sleep 1;
      done
## add this
    volumeMounts:
    - name: varlog
      mountPath: /var/log

  - name: sidecar
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/legacy-app.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}
## end
```

---

# 14 升級集群 (如何離線主機，並升級控制面板和升升級節點)
```
設置環境：  
kubectl config use-context k8s
```
Task  
現有的 kubernetes 集群正運行版本 1.20.0， **僅將主節點上** 的所有 kubernetes 控制平面和節點組升級到版本 1.20.1
確保在升之前 drain 主節點，並在升級後 uncordon 主節點
```
可使用以下命令通過 ssh 連接到主節點：
ssh mk8s-master-0

可使用以下命令在該主節點上獲取更高權限：
sudo -i
```
另外，在主節點上升級 kubelet 和 kubectl

## ANS:
Task/Administration with kubeadm/Upgrading kubeadm clusters
```
kubectl drain k8s-master-0 --ignore-daemonsets
apt update
apt install kubeadm=1.20.1-00 -y
kubeadm upgrade plan
kubeadm upgrade apply v1.20.1 -y --etcd-upgrade=false
kubectl uncordon k8s-master-0

apt install kubelet=1.20.1-00 kubectl=1.20.1-00

systemctl restart kubelet

kubectl get node
```

---

# 15 etcd 備份與恢復
```
設置環境：  
exit
```

Task  
為運行在 https://127.0.0.1:2379 上的現在 etcd 實例創建快照保存到 /data/backup/etcd-snapshot.db，
然後還原位於 /data/backup/etcd-snapshot-previous.db 的先前快照
```
提供以下 TLS 憑證和密鑰，以通過 etcdctl 連接服務器
CA cert: /opt/KUIN00601/ca.crt
client cert: /opt/KUIN00601/etcd-client.crt
client key: /opt/KUIN00601/etcd-client.key
```
## ANS:
Tasks/Administror a Cluster/Operating etcd cluster for kubenetes
```bash
# backup
ETCDCTL_API=3 etcdctl -h
snapshot
etcdctl snapshot save /data/backup/etcd-snapshot.db

# restore
systemctl stop etcd
etcdctl snapshot restore /data/bachup/etcd-snapshot.db

```

# 16 排查集群中故障節點
```
設置環境：  
kubectl config use-context  wk8s
```
名為 wk8s-node-0 的 kubernetes worker node 處於 NotReady 狀態，調查發生原因，並採取相應措施將 Node 恢復為 Ready
可用以下指令連到故障 node
```
ssh wk8s-node-0
sudo -i
```

# ANS:
依故障原因修改 kueblet 未啟動
```
systemctl status kubelet
systemctl start kubelet
systemctl enable kubelet 
```

# 17 節點維護 (cordon 與 drain 的使用)
```
設置環境：  
kubectl config use-context  ek8s
```

Task  
將名為 ek8s-node-1 的 node 設置為不可用，並重新調度該 node 上運行的 pods

## ANS:

```bash
kubelet cordon node ek8s-node-1
kubelet drain --ignort-daemonsets node ek8s-node-1
```