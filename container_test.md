# containerd

contaienrd 控制指令使用 `crictl`，在不同的 container 都可用 crictl 來控制。
crictl 的設定 /etc/crictl.yaml
```yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
image-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 2
debug: true
pull-image-on-create: false
```


# 一、 容器基礎
1. 在 vms100 上查看當前系統裏有多少鏡像
2. 在 vms100 上對 nginx:latest 做標籤，名字為 192.168.26.100/nginx:v1，並導出此鏡像為一個文件 niginx.tar
3. 在 vms100 上使用鏡像 192.168.26.100/nginx:v1 創建容器，滿足如下要求
    1. 容器名為 web
    2. 容器重啟策略設置為 always
    3. 把容器的端口 80 映射到物理機 (vms100) 的端口 8080 上
    4. 把物理機 (vms100) 目錄 /web 掛載到容器的 /usr/share/nginx/html 裏
4. 在容器 web 的目錄 /usr/share/nginx/html 裏創建文件 index.html，內容為 "hello docker"
5. 打開瀏覽器，地址欄輸入 192.168.26.100:8080，查看是否能看到 hello docker
6. 刪除容器 web 和鏡像 192.168.26.100/nginx:v1

---

# 二、 docker 進階

1. 在 vms100 上為了構建一個新的鏡像，請編寫一個 Dockerfile，要求如下：
    1. 基於鏡像 hub.c.163.com/library/contos:latest
    2. 新的鏡像裏包含 ifconfig 命令
    3. 新的鏡像裏包含變量 myname=test
    4. 新的鏡像裏包含一個用戶 tom，並且使用此鏡像容運行容器時，容器裏的進程以 tom 身份運行
    5. 使用此鏡像創建容器時，默認運行的進程為 /bin/bash
2. 使用此 Dockerfile 構建一個名字為 192.168.26.101:5000/cka/centos:v1 的鏡像
3. 在 vms101 上以容器的方式搭建一個本地私有倉庫，要求如下：
    1. 使用鏡像 hub.c.163.com/library/registry:latest
    2. 推送的鏡像要能持久保存在物理機 (vms101) 的 /myreg 目錄裏
    3. 容器名為 myreg
    4. 此容器的端口 5000 映射到物理機 (vms101) 的端口 5000 上
    5. 重啟策略設置為 always
4. 在 vms100 和 vms101 上適當修改配置，使得在 vms100 上不管是從 vms101 拉取鏡像，還是往 vms101 上推送鏡像，都以 http 的方式，而不是 https 的方式



