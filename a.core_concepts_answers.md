Create a namespace called 'mynamespace' and a pod with image nginx called nginx on this namespace

```
kubectl create ns mynamespace
kubectl run nginx --image=nginx -n mynamespace
```

Create the pod that was just described using YAML

```
kubectl run nginx --image=nginz -n mynamspace --dry-run=client -o yaml > nginx-mynamespace.yaml

$ cat nginx_mynamespace.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
  namespace: mynamespace
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


kubectl apply -f nginx_mynamespace.yaml
```
