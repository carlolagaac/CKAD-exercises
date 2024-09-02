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
kubectl run busybox --image=busybos --rm -it --restart=Never -- 


