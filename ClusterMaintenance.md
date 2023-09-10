# Cluster Maintenance

## OS Upgrade
document: Tasks/Administer a Cluster/Safely Drain a Node

in cluster have a pod error, how to doing
node dead 5 min, out of cluster, after add will empty node, no any pod

停止部署 pod 到指定 node 與取消，drain 會強制移走 pod 但 cordon 不會，如果不是用 deploy 的方式，單獨的 pod 會被刪除而永遠消失
```
kubectl drain node/ kubeclt cordon node
kubectl uncordon node
```
* 執行 node 上有 deamonset 會無法執行，可加上參數 --ignore-deamonsets


## Cluster Upgrade Process
document: Tasks/Administer a Cluster/Administration with kubeadm/Upgrading kubeadm clusters

Upgrading kubeadm clusters
kube-apiserver 元件使用版本規則：
* conrtoller-manager 與 kube-scheduler 可以低 apiserver 一個版本
* kubelet 與 kube-proxy 可以低 2個版本
* kubectl 可以高一個版本
* 最多只支援三個次版

kubectl upgrade plan
kubectl upgrade apply

master 與 node01 upgrade
1. 查看可用版本
```bash
apt-get madison kubeadm

kubeamd upgrade plan
```
2. 停用 master
假設目前 1.26 要升到 1.27
```bash
kubectl drain master --ignore-daemonsets
apt-get update
apt-get upgrade kubectl
apt-get install kubelet='1.27.0-00'
systemctl restart kubelet
kubectl uncordon master
```
3. 停用 node01
在 master node 下指令
```bash
kubectl drain node01
```
切到 node01
```bash
apt-get upgrade
apt-get upgrade kubelet
apt-get install kubele='1.27.0-00'
systemctl restart kubelet
```
切回 master
```bash
kubectl uncordon node01
```

## Backup And Restore

etcd 為記錄 k8s 所有狀態，可透過 etcdctl 輔助備份
etcdctl 只能透過 github 安裝
```bash
export ETCDCTL_API=3

etcdctl version
etcdctl snapshot save -h 

etcdctl snapshot restore -h
```

## multi cluster

kubectl config get-clusters

change cluster1
kubectl config use-context cluster1