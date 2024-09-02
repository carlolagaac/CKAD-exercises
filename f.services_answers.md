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