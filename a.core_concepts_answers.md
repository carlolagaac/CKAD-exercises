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


Create a busybox pod (using kubectl command) that runs the command "env". Run it and see the output
https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/


```
kubectl run busybox --iaage=busybox --command -- env
kubectl logs busybox
```

Create a busybox pod (using YAML) that runs the command "env". Run it and see the output 
```
kubectl run busybox --image=busybox  --dry-run=client  -o yaml --command -- env > busybox_env.yaml
```

Get the YAML for a new namespace called 'myns' without creating it

```
kubectl create ns myns --dry-run=client -o yaml > myns.ysml


Create the YAML for a new ResourceQuota called 'myrq' with hard limits of 1 CPU, 1G memory and 2 pods without creating it

```
kubectl create quota myrq --hard=limits.cpu=1,limits.memory=1Gi,count/pods=2 -o yaml

apiVersion: v1
kind: List
items:
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: myrq
  spec:
    hard:
      cpu: "1000"
      memory: 1Gi
      pods: "2"