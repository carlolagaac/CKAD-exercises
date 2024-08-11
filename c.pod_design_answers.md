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

Remove the annotations for these three pods
```
kubectl annotate po nginx{1..3} description- owner-
```


Remove these pods to have a clean state in your cluster
```
for i  in  1 2 3 
do
kubectl delete po nginx${i}
done
```

## Pod Placement

Create a pod that will be deployed to a Node that has the label 'accelerator=nvidia-tesla-p100'

```
kubectl label nodes <your-node-name> accelerator=nvidia-tesla-p100
kubectl get nodes --show-labels

kubectl run busybox --image=busybox --dry-run=client -o yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: accelerator
            operator: In
            values:
            - nvidia-tesla-p100

  containers:
  - image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}


# Taint a node with key tier and value frontend with the effect NoSchedule. Then, create a pod that tolerates this taint.
```
kubectl taint node k3d-ckad-exercises-server-1 tier=frontend:NoSchedule

apiVersion: v1
kind: Pod
metadata:
  name: frontend-tolerate
spec:
  containers:
  - name: frontend-tolerate
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
    - key: "tier"
      operator: "Equal"
      value: "frontend"
      effect: "NoSchedule"
```







