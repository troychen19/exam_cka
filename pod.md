# Test PODS

1. run pod
```shell
kubectl run nginx --image nginxs
```

2. run pod use YAML
| kind | Version |
| --- | --- |
| POD | v1 |
| Service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
      app: myapp
spec:
  containers
    - name: nginx-container
      image: nginx
```