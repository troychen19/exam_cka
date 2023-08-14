# 節點 taint 與 node 的 tolerations

**[重點]** 給節點設置/刪除 taint，設置 operator 的值為 Equal，以及設置 operator 的值為 Exists

當建立 pod 雖然 master 也是 ready，但 pod 並不會部署到 master。因為 master 有設置 taint (污點)。
當我們給某節點設置了 taint，只有那些設置了 tolerations 的 pod 可以運行在這些節點。

1. 查看 taint 的語法
```bash
  kubectl describe ndoes node01 | grep -E '(Roles|Taints)'
```
2. 設定節點 taint
設定節點 taint key=value 的語法為 key值=value值:effect ，一般 effect 為設為 NoSchedule 
    * 設定所有節點
    ```bash
      kubectl taint nodes -all key1=all:NoSchedule
    ```
    * 設定指定節點
    ```bash
      kubectl tain nodes node01 key1=node1:NoSchedule
    ```
3. 要將 pod 部署在設有 taint 的 node
需要在 yaml 中加入以下的設定，以下的設定表示 pod 會被部署到 taint 標註為 key1=node1 的節點
```yaml
tolerations:
- key: "key1" # key 的值
  operator: "Equal"
  value: "node1" # value 的值
```
operator 的值有兩種 Equal 和 Exists， Equal 的 key 與 value 需要和 node 相同，Exists 則不需指定 value 值
使用 Exists 時，設定 node 與 pods 設定檔都不能設定 value 