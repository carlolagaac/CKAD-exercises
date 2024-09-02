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
