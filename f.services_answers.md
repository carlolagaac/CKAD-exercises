## Services and Networking

Create a pod with image nginx called nginx and expose its port 80
```
kubectl run nginx --image=nginx --port=80 
kubectl expose po nginx --port=80
```

Confirm that ClusterIP has been created. Also check endpoints
```
kubectl get svc 
nginx        ClusterIP   10.43.7.54   <none>        80/TCP    55s

ubectl get ep
NAME         ENDPOINTS                                         AGE
nginx        10.42.2.75:80                                     4m42s
```

Get service's ClusterIP, create a temp busybox pod and 'hit' that IP with wget
```
kubectl run busybox --image=busybox --rm -it --restart=Never -- 
wget 10.42.2.75
Connecting to 10.42.2.75 (10.42.2.75:80)
saving to 'index.html'
```

Convert the ClusterIP to NodePort for the same service and find the NodePort port. Hit service using Node's IP. Delete the service and the pod at the end.
```
kubectl get svc nginx -o yaml > nginx-nodeport.yaml
kubectl edit svc nginx
  type: ClusterIP
  to 
  type: NodePort

kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
nginx        NodePort    10.43.7.54   <none>        80:30313/TCP   31m
```

