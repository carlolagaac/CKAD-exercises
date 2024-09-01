## Liveness, readiness and startup probes
Create an nginx pod with a liveness probe that just runs the command 'ls'. Save its YAML in pod.yaml. Run it, check its probe status, delete it.

```
kind: Pod
metadata:
  labels:
    test: nginx-liveness-test
  name: nginx-liveness
spec:
  containers:
  - name: nginx-liveness
    image: nginx
    livenessProbe:
      exec:
        command:
        - ls 
        - /tmp

kubectl get po nginx-liveness
kubectl describe po nginx-liveness | grep -i liveness
```

Modify the pod.yaml file so that liveness probe starts kicking in after 5 seconds whereas the interval between probes would be 5 seconds. Run it, check the probe, delete it.

```
kind: Pod
metadata:
  labels:
    test: nginx-liveness-test
  name: nginx-liveness
spec:
  containers:
  - name: nginx-liveness
    image: nginx
    livenessProbe:
      exec:
        command:
        - ls
        - /tmp
      initialDelaySeconds: 5
      periodSeconds: 5
```

Create an nginx pod (that includes port 80) with an HTTP readinessProbe on path '/' on port 80. Again, run it, check the readinessProbe, delete it.

```
kubectl run nginx-httpready --image=nginx --port=80 --dry-run=client -o yaml > nginx_httpready.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-httpready
  name: nginx-httpready
spec:
  containers:
  - image: nginx
    name: nginx-httpready
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```


Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.

```
kubectl get events -o json | jq -r '.items[] | select(.message | contains("failed liveness probe")).involvedObject | .namespace + "/" + .name'
kubectl get events -o json | jq -r '.items[] | select(.message | contains("Stopping")).involvedObject | .namespace + "/" + .name'
```

## Logging
Create a busybox pod that runs i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done. Check its logs
```
kubectl run busybox-while --image=busybox --command /bin/sh --dry-run=client -o yaml 

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox-while
  name: busybox-while
spec:
  containers:
  - command:
    - /bin/sh
    - "-c"
    - 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 5; done'
    image: busybox
    name: busybox-while
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## Debugging

Create a busybox pod that runs 'ls /notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod\
```
kubectl run busybox-notexist --image=busybox --command /bin/sh --dry-run=client -o yaml > busybox-notexist.yaml

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox-notexist
  name: busybox-notexist
spec:
  containers:
  - command:
    - /bin/sh
    - -c 
    - ls /notexist
    image: busybox
    name: busybox-notexist
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl get po
NAME               READY   STATUS      RESTARTS      AGE
busybox-notexist   0/1     Error       2 (20s ago)   30s
```


Create a busybox pod that runs 'notexist'. Determine if there's an error (of course there is), see it. In the end, delete the pod forcefully with a 0 grace period

```
kubectl run busybox-notexist2 --image=busybox --command notexist 
kubectl delete po busybox-notexist2 --force=true --grace-period=0
```




