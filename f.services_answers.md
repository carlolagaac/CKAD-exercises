## Services and Networking

Create a pod with image nginx called nginx and expose its port 80
```
kubectl run nginx --image=nginx --port=80 --expose
```

Confirm that ClusterIP has been created. Also check endpoints
```
kubectl get svc 
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx        ClusterIP   10.43.207.251   <none>        80/TCP    5s

kubectl get ep      
NAME         ENDPOINTS                                         AGE
nginx        10.42.2.83:80                                     85s

```

Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget
```
kubectl run busybox --rm --image=busybox -it --restart=Never --
wget 10.42.2.83
Connecting to 10.42.2.83 (10.42.2.83:80)
saving to 'index.html'
/ # exit
pod "busybox" deleted
```

Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.
```
kubectl edit svc nginx
change
 type: ClusterIP
to
 type: NodePort

ubectl get svc                                                
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
nginx        NodePort    10.43.207.251   <none>        80:30234/TCP   18m
```

Create a deployment called foo using image 'dgkanatsios/simpleapp' (a simple server that returns hostname) and 3 replicas. Label it as 'app=foo'. Declare that containers in this pod will accept traffic on port 8080 (do NOT create a service yet)

```
kubectl create deploy foo --image=dgkanatsios/simpleapp --replicas=3 --port=8080 --dry-run=client 
```

Get the pod IPs. Create a temp busybox pod and try hitting them on port 8080
```
kubectl run busybox --image=busybox --rm -it --restart=Never -- 

wget 10.42.0.27:8080
Connecting to 10.42.0.27:8080 (10.42.0.27:8080)
saving to 'index.html'
index.html           100% |*******************************************************************************************************|    54  0:00:00 ETA
'index.html' saved
```


Create a service that exposes the deployment on port 6262. Verify its existence, check the endpoints
```
kubectl expose deployment nginx --port=6262 --target-port=8080
kubctl get ep 
kubectl get ep
NAME         ENDPOINTS                                         AGE
foo          10.42.0.27:8080,10.42.1.24:8080,10.42.2.60:8080   20s
```

Create a temp busybox pod and connect via wget to foo service. Verify that each time there's a different hostname returned. Delete deployment and services to cleanup the cluster


```
kubectl run busybox --image=busybox --rm -it --restart=Never -- 
If you don't see a command prompt, try pressing enter.
/ # 
/ # 
/ # wget -c 10.43.29.134:6262
Connecting to 10.43.29.134:6262 (10.43.29.134:6262)
saving to 'index.html'
index.html           100% |**********************************************************************************************************|    54  0:00:00 ETA
'index.html' saved
/ # more index.html
Hello world from foo-5566d9bc7c-wdx9d and version 2.0
Hello world from foo-5566d9bc7c-w8bfm and version 2.0
Hello world from foo-5566d9bc7c-wxrmc and version 2.0

kubectl delete svc foo
kubectl delete deploy foo
```

Create an nginx deployment of 2 replicas, expose it via a ClusterIP service on port 80. Create a NetworkPolicy so that only pods with labels 'access: granted' can access the deployment and apply it

```
kubectl create deploy nginx-deploy --image=nginx --replicas=2 --port=80
kubectl expose deploy nginx-deploy
kubectl label deploy nginx-deploy access=granted

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: granted-pods
  
spec:
  podSelector:
    matchLabels:
      access: granted
  policyTypes:
  - Ingress
  ingress: 
  - from: 
    - podSelector:
        matchLabels:
          access: granted

kubectl get svc 
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                        PORT(S)          AGE
nginx-deploy   ClusterIP      10.43.180.33    <none>                             80/TCP           9m24s

kubectl get ep
NAME           ENDPOINTS                                               AGE
nginx-deploy   10.42.1.58:80,10.42.2.62:80                             10m


kubectl run busybox --image=busybox --labels=access=granted --rm -it --restart=Never -- 

wget -c 10.43.180.33
Connecting to 10.43.180.33 (10.43.180.33:80)
saving to 'index.html'
index.html           100% |*************************************************************************************************|   615  0:00:00 ETA
'index.html' saved

```

 