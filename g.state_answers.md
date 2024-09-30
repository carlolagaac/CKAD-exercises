# Persistence
## Define Volumes

Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

```
kubectl run busybox --image=busybox  --dry-run=client -o yaml  --command -- '/bin/sh' '-c' 'sleep 300'

apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - sleep 300
    image: busybox
    name: busybox1
    volumeMounts:
    - mountPath: /etc/foo
      name: foo-volume

  - command:
    - /bin/sh
    - -c
    - sleep 300
    image: busybox
    name: busybox2
    volumeMounts:
    - mountPath: /etc/foo
      name: foo-volume

  volumes:
  - name: foo-volume
    emptyDir:
      sizeLimit: 500Mi

  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}



kubectl exec busybox -c busybox1 -it -- /bin/sh 
cp /etc/passwd /etc/foo/passwd

kubectl exec busybox -c busybox2 -it -- /bin/sh 
cat /etc/foo/passwd

```

Create a PersistentVolume of 10Gi, called 'myvolume'. Make it have accessMode of 'ReadWriteOnce' and 'ReadWriteMany', storageClassName 'normal', mounted on hostPath '/etc/foo'. Save it on pv.yaml, add it to the cluster. Show the PersistentVolumes that exist on the cluster



