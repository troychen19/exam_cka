# 1. We have created a new deployment called `nginx-deploy`. scale the deployment to `3 replicas`. Has the replica's increased? Troubleshoot the issue and fix it.

## ANS
Use the command kubectl scale to increase the replica count to 3.  
```
kubectl scale deploy nginx-deploy --replicas=3
```
The `controller-manager` is responsible for scaling up pods of a replicaset. If you inspect the control plane components in the `kube-system` namespace, you will see that the controller-manager is not running.   
```
kubectl get pods -n kube-system
```
The command running inside the controller-manager pod is `incorrect`.
After fix all the values in the file and wait for controller-manager pod to restart.

Alternatively, you can run sed command to change all values at once:

sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
This will fix the issues in controller-manager yaml file.

At last, inspect the deployment by using below command:

kubectl get deploy

---

# 2. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.
Next, create a pod called pvviewer with the image: redis and serviceAccount: pvviewer in the default namespace.

## ANS:
Pods authenticate to the API Server using ServiceAccounts. If the serviceAccount name is not specified, the default service account for the namespace is used during a pod creation.  

Reference: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/  

Now, create a service account pvviewer:
```
kubectl create serviceaccount pvviewer
```
To create a clusterrole:
```
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
```
To create a clusterrolebinding:
```
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer
```

Solution manifest file to create a new pod called pvviewer as follows:
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    name: pvviewer
  # Add service account name
  serviceAccountName: pvviewer
```


---

# 3. List the InternalIP of all nodes of the cluster. Save the result to a file /root/CKA/node_ips.

## ANS:
Answer should be in the format: InternalIP of controlplane<space>InternalIP of node01 (in a single line)
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}' > /root/CKA/node_ips
```

---

# 4. We have deployed a new pod called np-test-1 and a service called np-test-service. Incoming connections to this service are not working. Troubleshoot and fix it.
Create NetworkPolicy, by the name ingress-to-nptest that allows incoming connections to the service over port 80.  

Important: Don't delete any current objects deployed.

Solution manifest file to create a network policy ingress-to-nptest as follows:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
```

---

# 5. Taint the worker node node01 to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine, to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image: redis:alpine with toleration to be scheduled on node01.

## ANS
`key: env_type, value: production, operator: Equal and effect: NoSchedule`

To add taints on the node01 worker node:  
```
kubectl taint node node01 env_type=production:NoSchedule
```

Now, deploy dev-redis pod and to ensure that workloads are not scheduled to this node01 worker node.
```
kubectl run dev-redis --image=redis:alpine
```
To view the node name of recently deployed pod:
```
kubectl get pods -o wide
```

Solution manifest file to deploy new pod called prod-redis with toleration to be scheduled on node01 worker node.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  containers:
  - name: prod-redis
    image: redis:alpine
  tolerations:
  - effect: NoSchedule
    key: env_type
    operator: Equal
    value: production     
```

To view only prod-redis pod with less details:
```
kubectl get pods -o wide | grep prod-redis
```