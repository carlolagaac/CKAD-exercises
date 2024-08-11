# Pod design (20%)

## Labels and Annotations

Create 3 pods with names nginx1,nginx2,nginx3. All of them should have the label app=v1

```
kubectl run nginx1 --image=nginx --labels="app=v1" --dry-run=client -o yaml 
kubectl run nginx2 --image=nginx --labels="app=v1" --dry-run=client -o yaml 
kubectl run nginx3 --image=nginx --labels="app=v1" 
```

Show all labels 
```
kubectl get po --show-labels
```

Change the labels of pod 'nginx2' to be app=v2
```
kubectl label po nginx2 "app=v2" --overwrite
```

Get the label 'app' for the pods (show a column with APP labels)
```
kubectl get po -L app  
```


Get only the 'app=v2' pods
```
kubectl get po -l app=v2
kubectl get po --selector=app=v2
```

Add a new label tier=web to all pods having 'app=v2' or 'app=v1' labels
```
kubectl label po -l "app in (v1,v2)" tier=web
```

Add an annotation 'owner: marketing' to all pods having 'app=v2' label
```
kubectl annotate po -l app=v2 owner=marketing
```

Remove the 'app' label from the pods we created before
```
kubectl label po  -l app app-
```

Annotate pods nginx1, nginx2, nginx3 with "description='my description'" value
```
kubectl annotate po nginx{1..3} description="my description"
```
kubectl annotate pod nginx1 --list
kubectl describe po nginx1 | grep -i annotations
```

Remove these pods to have a clean state in your cluster
```
for i  in  1 2 3 
do
kubectl delete po nginx${i}
done
```







