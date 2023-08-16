# 一、 emptyDir
**重點** 創建 emptyDir 類型的 volume，並掛載
使用 emptyDir 就如同 docker 容器的命令 docker run -v /xx，意思就是在物理機隨機產生一個目錄，然把這個目錄掛載到容器的 /xx 目錄。當 pod 刪除時 emptyDir 目錄也會一併刪除。
1. 建立 volume 目錄 `mkdir volume`
2. 建立 yaml 檔
```bash
  kubectl run demo --dry-run=client --image=busybox -o yaml > emp.yaml
```
3. 修改 yaml 
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: demo
  name: demo
spec:
  volumes:
  # 第一個 volume
  - name: volume1
    emptyDir: {}
  # 第二個 volume
  - name: volume2
    emptyDir: {}
  containers:
  - name: demo1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    # volume1 掛載到 /v1
    volumeMounts:
    - mountPath: /v1
      name: volume1

  - name: demo2
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    # volume1 掛載到 /v2
    volumeMounts:
    - mountPath: /v2
      name: volume1

```
使用 `kubectl describe` 檢查 pod 可看到 /v1, /v2 都掛載到了 /volume1

查看 demo1 與 demo2 的目錄
```bash
  kubectl exec demo -c demo1 -- ls /v1

  kubectl exec demo -c demo2 -- ls /v2
```

# 二、 hostPath
**重點** 定義 hostPath 類型的儲存
使用 hostPath 類似使用 `docker run -v /data:/xx`，意思是在 host 上建立 /data 目錄，然後將 /data 映射到容器的 /xx 目錄
定義方式如下：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    type: demo-host
spec:
  # 創建一個名為 volume1 的 hostPath
  volumes:
  - name: volume1
    hostPath:
      path: /data
  containers:
  - name: demo1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh', '-c','sleep 5000']
    # 將 volume1 掛載到 /v1
    volumeMounts:
    - mountPath: /v1
      name: volume1

  - name: demo2
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    # 將 volume1 掛載到 /v2
    volumeMounts:
    - mountPath: /v2
      name: volume1

```

檢查是否建立成功
1. 在 demo2 建立檔案 readme
```bash
  kubectl exec demo -c demo2 -- touch /v2/readme
```
2. 檢查是否在 demo1 出現
```bash
  kubectl exec demo -c demo1 -- ls /v1
```
3. 在實體機器上檢查 /data 目錄 (要在部署的主機上檢查)


# 三、 使用 NFS
**重點** 創建 nfs 類型 volume
不管是 emptyDir 還是 hostPath，雖然 Pod 間可以共用，但依然只存在各別 node 上，無法跨 node 使用。
1. 建立 NFS service
[Install NFS Server in Ubuntu22](https://linuxhint.com/install-and-configure-nfs-server-ubuntu-22-04/)
2. 建立 nfs.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
  labels:
    run: nginx
spec:
  volumes:
  - name: volume1
    nfs:
     server: 192.168.153.101
     path: /mnt/nfs_share
  containers:
  - name: demo1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    volumeMounts:
    - name: volume1
      mountPath: /v_nfs
```

# 四、 配置持久性儲存
**重點** 建立/刪除持久性存儲 (pv)，建立/刪除持久性存儲聲明 (pvc)

Persisent Volume (pv) 與指定的後端存儲關聯，需要專人指定，不屬於任何 namespace，全域可見。
1. 查看 pv
```bash
  kubectl get pv
```
2. 建立 pv
注意 storage 大小和 accessMode 的值，這個是 pvc 和 pv 綁定的關鍵
accessMode 有三種
* ReadWriteOnce (RWO)：允許單個節點掛載讀寫
* ReadOnlyMany (ROX)：允許多個節點掛載讀取
* ReadWriteMany (RWX)：允許多個節點掛載讀寫
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  # 指定容量
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  # 訪問模式
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  # 存儲的類型
  nfs:
    path: /mnt/nfs_share
    server: 192.168.153.101
```
3.  建立 pvc (PersistentVolumeClaim)
pvc 是基於 namespace 創建的，不同 namespace 的 pvc 相互隔離。通過 storage 的大小與 accessMode 與 pv 綁定，storage 的大小不能超過 pv 的 storage。
查看 pvc
```bash
  kubectl get pvc
```
建立 pvc
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc01
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Gi
```
查看建立的 pvc1
```bash
# /home/vagrant/volume# kubectl get pvc
NAME    STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc01   Bound    pv1      10Gi       RWO                           8s
```
4. pv 和 pvc 還可能透過 storageClassName 綁定
persistentVolumeReclaimPolicy 定義了回收策略：
* Recycle：刪除 pvc 後會生成一個 pod 回收刪除 pv裏的資料，刪除後 pv 可再被使用
* Retain：不刪除資料，刪除 pvc 後，pv 依然不可使用，需要刪除 pv 後重建，刪除 pv 不會刪除裏面的值
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  storageClassName: nfs_share
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    path: /mnt/nfs_share
    server: 192.168.153.101
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc01
spec:
  storageClassName: nfs_share
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi
```
5. 在 Pod 中使用
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc_demo
  labels:
    run: busybox
spec:
  volumes:
  - name: mypv
    persistentVolumeClaim:
      claimName: pvc01
  containers:
  - name: demo1
    image: busybox
    imagePullPolicy: IfNotPresent
    command: ['sh','-c','sleep 5000']
    volumeMounts:
    - name: mypv
      mountPath: /pv1
```

# 五、 配置動態 volume