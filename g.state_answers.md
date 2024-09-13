# Persistence
## Define Volumes

Create busybox pod with two containers, each one will have the image busybox and will run the 'sleep 3600' command. Make both containers mount an emptyDir at '/etc/foo'. Connect to the second busybox, write the first column of '/etc/passwd' file to '/etc/foo/passwd'. Connect to the first busybox and write '/etc/foo/passwd' file to standard output. Delete pod.

```
kubectl run busybox --image=busybox  --dry-run=client -o yaml  --command -- '/bin/sh' '-c' 'sleep 300'
