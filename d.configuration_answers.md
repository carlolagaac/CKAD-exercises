Create a configmap named config with values foo=lala,foo2=lolo
```
kubectl create configmap config --from-literal=foo=lala --from-literal=foo2=lolo
```

Display its values
```
kubectl get configmap config
```

Create and display a configmap from a file
```
kubectl create configmap my-config --from-file=./config.txt
kubectl get configmap my-config -o yaml 
```

