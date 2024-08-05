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
```

Get pods on all namespaces

```
kubectl get po -A
```

Create a pod with image nginx called nginx and expose traffic on port 80

```
kubectl run nginx --image=nginx 
kubectl expose po nginx --port 80
```

Change pod's image to nginx:1.26.1 Observe that the container will be restarted as soon as the image gets pulled
```
kubectl set image pod/nginx nginx=nginx:1.26.1
```

Get nginx pod's ip created in previous step, use a temp busybox image to wget its '/'

kubectl get po -o wide 
kubectl run busybox --image=busybox --rm -it --restart=Never -- wget -O- 10.1.1.131:8

Get nginx pod's YAML
```
kubectl get po nginx -o yaml
```

Get information about the pod, including details about potential issues (e.g. pod hasn't started)
```
kubectl describe po nginx
```

Get pod logs
```
kubectl logs nginx
```

Execute a simple shell on the nginx pod
```
 kubectl exec nginx  -it -- /bin/sh
```

Create a busybox pod that echoes 'hello world' and then exits
```
kubectl run busybox --image=busybox -it --restart=Never -- echo "hello world"
```

Do the same, but have the pod deleted automatically when it's completed
```
kubectl run busybox --image=busybox --rm -it --restart=Never -- echo "hello world"
```

Create an nginx pod and set an env value as 'var1=val1'. Check the env value existence within the pod
```
kubectl run nginx --image=nginx --env var1=val1
kubectl exec nginx -it /bin/sh 
env