# 1. 測試

## 1.1 使用環境
在 Ubuntu 20.04 下建 k8s 1.26.5 的測試環境 
使用 containerd 做為 k8s 的容器，並使用 docker 管理 image

## 1.2 初始化並加入 worker node
```shell
sudo kubeadm init --control-plane-endpoint="master"
```
使用 flannel 在初始化的時候要指定 pod-network-cidr  
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```


查看 container 的指令
```shell
sudo crictl -r unix:///run/containerd/containerd.sock images ls
```


## 1.3 使用 CNI 並確認所有的 ndoe ready

1. **Calico**
* download yaml
```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
```
* apply
```
kubectl apply -f ./calico.yaml
```

2. **Flannel**
* download yaml
```
curl https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml -O -L
```
* apply
```
kubectl apply -f ./kube-flannel.yaml
```

友善工具
kubecontext + kubens

# 2. 考試相關說明

* [考試報名流程](https://blog.51cto.com/u_7075695/3175461)
* [中文報名網站](https://training.linuxfoundation.cn/)
* [英文報名網站](https://trainingportal.linuxfoundation.org)
* [考試註冊](https://trainingportal.linuxfoundation.org/courses/certified-kubernetes-administrator-china-exam-cka-cn)
* [考試環境檢查](https://syscheck.bridge.psiexams.com/)
* [考古題](https://www.vcesoft.com/)

考試中可查閱的文件
* https://kubernetes.io/docs/concepts/
* https://kubernetes.io/docs/tasks/
* https://kubernetes.io/docs/reference/

# 2.1 考生手冊
https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad

# 2.2 相關考試心得

* [CKA 考試全攻略流程](https://medium.com/@app0/cka-%E8%80%83%E8%A9%A6%E5%85%A8%E6%94%BB%E7%95%A5%E6%B5%81%E7%A8%8B-3a28d1b73eea)
* [CKA認證考試就看這一篇](https://blog.csdn.net/mianbaojiayou/article/details/122449874)
* [手把手帶你過CNCF CKA考試 (第一章)](./handTohandCKA_first.md)
* [20221214 CKA (Certified Kubernetes Administrator) 考試心得](https://ithelp.ithome.com.tw/articles/10310401)
* [Curriculum](https://github.com/cncf/curriculum/tree/master)

# 2.3 CKA 模擬考題

* [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
* [CKA Exercises](https://github.com/chadmcrowell/CKA-Exercises/blob/main/README.md)
* [CKA preparation](https://github.com/alijahnas/CKA-practice-exercises/tree/CKA-v1.23)
* [Kubernetes Certified Administration](https://github.com/walidshaari/Kubernetes-Certified-Administrator)
* [Certified Kubernetes Administrator (CKA) Course](https://github.com/kodekloudhub/certified-kubernetes-administrator-course)

# 3. 教學課程

* [Certified Kubernetes Administrator (CKA) 考试完全指南（2022版）](https://www.udemy.com/course/k8s-chinese/)
* [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)
* [Udemy Labs – Certified Kubernetes Administrator with Practice Tests](https://kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/?utm_source=udemy&utm_medium=labs&utm_campaign=kubernetes)


# 4. [容器模擬題](./container_test.md)
# 5. [cluster安裝與配置](./cluster_deployAndConfig.md)
# 5.1 [升版與備份](./ClusterMaintenance.md)
# 6. [yaml 說明](./yaml.md)
# 7. [pod](./pod.md)
# 8. [schedule](./schedule.md)
# 9. [存儲管理](./storage.md)
# 10. [密碼與參數管理](./securityAndConfigMap.md.md)
# 11. [deployment 與 Lifecycle Management](./deployment.md)
# 12. [daemonset 及其它控制器](./daemontsetAndController.md)
# 13. [探針](./liveness.md)
# 14. [job](job.md)
# 15. [服務管理](./service.md)
# 16. [網路管理](./networking.md)
# 17. [安全管理](./SecurityAndrole.md)