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

Create a pod that will be placed on node controlplane. Use nodeSelector and tolerations.

```
kubectl run nginx-controlplane --image=nginx --dry-run=client -o yaml > nginx-controlplane.yaml
```

### Deployments

Create a deployment with image nginx:1.18.0, called nginx, having 2 replicas, defining port 80 as the port that this container exposes (don't create a service for this deployment)
```
kubectl create deployment nginx --image=nginx --replicase=2 --port=80
kubectl create deploy nginx --image=nginx --replicas=2 --port=80 --dry-run=client -o yaml


```

View the YAML of this deployment

```
kubectl get deploy nginx -o yaml
```


Check how the deployment rollout is going
```
kubectl rollout history nginx 
kubectl rollout status  deployment nginx 
```


Update the nginx image to nginx:1.26.1
```
kubectl create deploy nginx --image=nginx --replicas=2 --port=80 --dry-run=client -o yaml > nginx_deploy.yaml
kubectl replace -f nginx_deploy.yaml
``` 

Check the rollout history and confirm that the replicas are OK
```
kubectl rollout status  deployment nginx 
kubectl rollout history deploy nginx
kubectl rollout undo deployment nginx
```

Do an on purpose update of the deployment with a wrong image nginx:1.91
```
kubectl set image deploy nginx nginx=nginx:1.91
kubectl rollout status  deployment nginx   
```

Verify that something's wrong with the rollout
```
kubectl rollout status deploy nginx
kubectl get po -A
kubectl describe po nginx-7bb5748555-gpd4r
```

Return the deployment to the second revision (number 2) and verify the image is nginx:1.19.8

```
kubectl rollout history deploy nginx
kubectl rollout undo deploy/nginx --to-revision=2
```

Check the details of the fourth revision (number 4)
```
kubectl rollout history deploy/nginx --revision=4
```

Scale the deployment to 5 replicas
```
kubectl scale deploy/nginx --replicas=5
```

Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%
```
kubectl autoscale deploy/nginx --min=2 --max=10 --cpu-percent=80
```

Pause the rollout of the deployment
``` 
kubectl rollout pause deploy/nginx
```

Update the image to nginx:1.19.9 and check that there's nothing going on, since we paused the rollout
```
kubectl set image deploy/nginx nginx=nginx:1.9.1
```

Resume the rollout and check that the nginx:1.19.9 image has been applied
```
kubectl rollout resume deploy/nginx
kubectl describe po nginx-7ffd5c8dc9-wpdqc | grep nginx:1.9.1
```

Delete the deployment and the horizontal pod autoscaler you created
```
kubectl delete deploy nginx
kubectl get hpa
kubectl delete hpa nginx
```

Implement canary deployment by running two instances of nginx marked as version=v1 and version=v2 so that the load is balanced at 75%-25% ratio
```
#kubectl create deploy my-app-v1 --image=nginx --replicas=3 --dry-run=client -o yaml > my-app-v1.yaml 
kubectl create deploy my-app-v1 --image=nginx --replicas=3
kubectl label deploy my-app-v1  version=v1
kubectl get deploy/my-app-v1 -o yaml > my-app-v1.yaml
next edit the app label to :
app: my-app
```

in both files

```
kubectl create service loadbalancer my-app --tcp=8080:80 
```

# Jobs
Create a job named pi with image perl:5.34 that runs the command with arguments "perl -Mbignum=bpi -wle 'print bpi(2000)'"
```
kubectl create job pi --image=perl:5.34 -- perl -Mbignum=bpi -wle 'print bpi(2000)'
```

Wait till it's done, get the output
```
kubectl describe job pi
kubectl get po
kubectl logs pi-jln5q 
```

Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'
```
kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
kubectl get job
kubectl logs busybox-vlpwn
```

Delete the job
```
kubectl delete job busybox
```

Create a job but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute
```
kubectl create job 30secs --image busybox --dry-run=client -o yaml -- /bin/sh "-c" "while true;do; sleep 1; echo hi;done" > 30secs.yaml
add to yaml
spec:
  activeDeadlineSeconds: 30
```

Create the same job, make it run 5 times, one after the other. Verify its status and delete it
```
kubectl create job 30secs --image busybox --dry-run=client -o yaml -- /bin/sh "-c" "while true;do; sleep 1; echo hi;done" > 30secs.yaml
add to yaml
spec:
  activeDeadlineSeconds: 30
  completions: 5

Create the same job, but make it run 5 parallel times
``` 
kubectl create job 30secs --image busybox --dry-run=client -o yaml -- /bin/sh "-c" "while true;do; sleep 1; echo hi;done" > 30secs.yaml
add to yaml
spec:
  parallelism: 5
  completions: 5


## Cron jobs







