# 手把手帶你過CNCF CKA考試 (第一章)

**簡介** ： 隨著雲原生和golang的日漸發展,CKA的普及度也越來越高了.現在市面上均是收費題庫.並沒有太多整體的複習資料,本人於2021年5月剛剛通過CKA考試.本文將分享我的考試心得及複習資料

**先說心得**

---

* 每道題請務必看是否需要執行 `kubectl config use-context k8s`來載入環境變數,只有少部分的題不需要載入環境變數(因為可能用的是上一題的環境).環境變數名稱大部分是 `hk8s` `mk8s` 等等.環境變數會直接導致你答案的精準性.
* 不會的題可以點選左下側按鈕記性 flag 標記等全部完成後再回來做題(再次提醒如果是這種情況也要務必記得重新載入本題的環境變數)
* 故障排查問題/叢集升級問題 需要進入對應節點 提權至 root 權限後進行組態,等本題操作完成後,務必記得退出到 student(本地)的控制台再進行下一題,(需要退出兩次,第一次是退出到非 root 帳戶, 第二次是退出對應節點ssh)
* 考試時允許開啟最多一個tab頁面來查閱文件,可以提前在我的最愛裡把一些重點文件保存下來方便查閱(.io的搜尋系統有時候不好用需要多請求幾次)

**考試大綱**

---

25% - Cluster Architecture, Installation & Configuration
---
• Manage role based access control (RBAC)
• Use Kubeadm to install a basic cluster
• Manage a highly-available Kubernetes cluster
• Provision underlying infrastructure to deploy a Kubernetes cluster
• Perform a version upgrade on a Kubernetes cluster using Kubeadm
• Implement etcd backup and restore

---

*  RBAC的管理 [參考文件](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
Rolebinding不能跨namespace, ClusterRolebinding可以跨namespace,考察RBAC，要求建立指定namespace下的serviceAccount，並建立 Role/ClusterRole，繫結serviceAccount。要求指定的sa對指定namespace有建立pod的權限（如果是對整個叢集的話，需要用ClusterRoleBinding）
```shell
# 建立一個namespace
kubectl create namespace app-team1
# 建立一個serviceaccount 到 app-team1 這個namespace下面 名稱是cicd-token
kubectl -n app-team1 create serviceaccount cicd-token
# 建立一個跨叢集的繫結關係
kubectl -n app-team1 create rolebinding cicd-token-binding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token
# 查看這個繫結關係
kubectl -n app-team1 describe rolebindings.rbac.authorization.k8s.io cicd-token-binding
```

* 使用Kubeadm 安裝基礎叢集 (參考文件)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/]
現在基本上都是可以通過 yum/apt 等直接安裝 kubeadm,考試應該是 debian 所以使用 apt-get 安裝 kubeadm工具.安裝完後直接執行 `kubeadm init` 即可

* 管理高可用的k8s叢集 (參考文件)[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/]
考試一般都會提供你k8s環境 你直接use 環境變數即可引用相關內容,在這裡不做過多介紹.部署一個高可用的叢集優先要保證各個節點的 etcd 都在運行後參考文件連結即可完成部署.