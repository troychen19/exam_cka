# Job

---
**重點**  
1. 建立刪除 job
2. 建立刪除 cronjob
---

在日常工作中會遇到一些執行完成後就不需要執行的程序，如分析、測試，結束後不需一直運行。也可有定時清 temp 檔...，需要定時執行的工作，可以透過 job 與 cronjob 來實現。

## job  
1. 建立 job yaml 檔
   ```bash
     kubect create job job1 --image=buxybox --dry-run=client -o yaml -- sh -c "echo hello && sleep 10" > job1.yaml
   ```
   生成的 yaml 檔如下：
   ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
    creationTimestamp: null
    name: job1
    spec:
    template:
        metadata:
        creationTimestamp: null
        spec:
        containers:
        - command:
            - sh
            - -c
            - echo hello && sleep 10
            image: busybox
            name: job1
            resources: {}
        restartPolicy: Never
    status: {}
   ```
2. 執行 job  
   ```bash
    kubect apply -f job1.yaml
   ```
   查看有一個 job 在執行
   ```bash
    kubectl get jobs
   ```
   查看 pods 的 status 為 complete
   ```bash
    # kubect get pods
    NAME                         READY   STATUS      RESTARTS   AGE
    job1-2fls4                   0/1     Completed   0          26s
 
   ```
3. job 中可指定參數  
job 因為是一次性的任務，所以 job 因為各種原因無法正確執行時，可透過以下參數調整
   * parallelism： N，並行在 N 個 pod
   * completions： M，job 測試多次的話，要有 M 次成功才算成功，即要有 M 個 complete 的 pod，没有就重複執行
   * backoffLimit：N，如果 job 失敗則重複 N 次 (有些 pod 完成後會被刪除，因此不能看最終 pod 數)
   * activeDeadlineSeconds：N，job 運行最長時間，單位是秒
以下為調整後的 job，平行跑 3 個 pod ，要 6 pod complete 才成功，
   ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
    creationTimestamp: null
    name: job1
    spec:
    parallelism: 3
    completions: 6
    backoffLimit: 4
    template:
        metadata:
        creationTimestamp: null
        spec:
        containers:
        - command:
            - sh
            - -c
            - echo hello && sleep 10
            image: busybox
            name: job1
            resources: {}
        restartPolicy: Never
    status: {}
   ```

## cronjob

cronjob (簡稱 cj) 為週期性作業，設定格式同 linux 的 crontab
1. 建立 cronjob 的 yaml 檔
   ```bash
    kubectl create cronjob job2 --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- /bin/sh -c "echo hello world" > job2.yaml
   ```
   建立後的檔案內容如下：
   ```yaml
    apiVersion: batch/v1
    kind: CronJob
    metadata:
    creationTimestamp: null
    name: job2
    spec:
    jobTemplate:
        metadata:
        creationTimestamp: null
        name: job2
        spec:
        template:
            metadata:
            creationTimestamp: null
            spec:
            containers:
            - command:
                - /bin/sh
                - -c
                - echo hello world
                image: busybox
                name: job2
                resources: {}
            restartPolicy: OnFailure
    schedule: '*/1 * * * *'
    status: {}
   ```
2. 建立 cronjob
   ```bash
    kubectl apply -f job2.yaml
   ```
3. 查看
   ```bash
    # kubectl get cj
    NAME   SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
    job2   */1 * * * *   False     0        <none>          6s
   ```