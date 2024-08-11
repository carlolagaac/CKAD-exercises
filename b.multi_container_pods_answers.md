Create a Pod with two containers, both with image busybox and command "echo hello; sleep 3600". Connect to the second container and run 'ls'

```
kubectl run busybox --image busybox --dry-run=client -o yaml --command "/bin/sh" > busybox.pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: busybox
spec:
  containers:
  - command: ["/bin/sh", "-c"]
    args: ["echo hello; sleep 3600"]
    image: busybox
    name: busybox
    resources: {}
  - command: ["/bin/sh", "-c"]
    args: ["echo hello; sleep 3600"]
    image: busybox
    name: busybox2
    resources: {}


kubectl exec busybox -c busybox2 -it -- /bin/sh
ls
exit
```

Create a pod with an nginx container exposed on port 80. Add a busybox init container which downloads a page using "wget -O /work-dir/index.html http://neverssl.com/online". Make a volume of type emptyDir and mount it in both containers. For the nginx container, mount it on "/usr/share/nginx/html" and for the initcontainer, mount it on "/work-dir". When done, get the IP of the created pod and create a busybox pod and run "wget -O- IP"

```
kubectl run nginx --image nginx --port 80 
kubectl run busybox