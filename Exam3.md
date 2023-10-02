# 1. Create a Kubernetes secret as follows:  
* Name: suepr-secret
* Password: bob
Create a pod name `pod-secrets-via-file`, use the `redis` image, which mount a secret name `super-secret` at /secrets
Create a second pod name `pod-secrets-via-env`, use the `redis` image, which exports password as CONFIDENTIAL

## ANS

1. create securite
```
kubectl create sercret generic super-sercet --from-literal=password=bob
```
2. create pod-secrets-via-file
```yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secrets-via-file
spec:
  containers:
  - name: redis
    image: redis
    voluemMounts:
    - name: foo
      mountPath: "/secrets"
  volumes:
  - name: foo
    secret:
      secretname: suepr-secret
```
3. create pod-secrets-via-env
```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: vpod-secrets-via-file
    spec:
      containers:
      - name: redis
        image:  mysql:redis
        imagePullPolicy: IfNotPresent
      env:
        - name: CONFIDENTIAL
          valueFrom:
            secretKeyRef:
              name: super-secret
              key: password
```

---

# 2.