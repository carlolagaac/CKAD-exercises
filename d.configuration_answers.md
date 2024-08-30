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

Create and display a configmap from a .env file
```
echo -e "var1=val1\n# this is a comment\n\nvar2=val2\n#anothercomment" > config.env
kubectl create configmap config-env --from-file=./config.env
kubectl get configmap config-env -o yaml
```

Create a configMap called 'options' with the value var5=val5. Create a new nginx pod that loads the value from variable 'var5' in an env variable called 'option'

```
kubectl create configmap options --from-literal=var5=val5

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: option
          valueFrom:
            configMapKeyRef:
              name: options
              key: var5

  restartPolicy: Never

```

Create a configMap 'anotherone' with values 'var6=val6', 'var7=val7'. Load this configMap as env variables into a new nginx pod
```
kubectl create configmap anotherone --from-literal=var6=val6 --from-literal=var7=val7

apiVersion: v1
kind: Pod
metadata:
  name: nginx-anotherone
spec:
  containers:
    - name: nginx
      image: nginx
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: option1
          valueFrom:
            configMapKeyRef:
              name: anotherone
              key: var6

        - name: option2
          valueFrom:
            configMapKeyRef:
              name: anotherone
              key: var7

  restartPolicy: Never
```


Create a configMap 'cmvolume' with values 'var8=val8', 'var9=val9'. Load this as a volume inside an nginx pod on path '/etc/lala'. Create the pod and 'ls' into the '/etc/lala' directory

```
kubectl create configmap cmvolume --from-literal=var8=val8 --from-literal=var9=val9
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cm-vol
spec:
  containers:
    - name: nginx-cm
      image: nginx
      command: ['sh', '-c', 'echo "The app is running!" && tail -f /dev/null']
      volumeMounts:
        - name: configmap-vol
          mountPath: /etc/lala
  volumes:
    - name: configmap-vol
      configMap:
        name: cmvolume

kubectl exec nginx-cm-vol -it -- /bin/sh
df 
```

## Security Context
Create the YAML for an nginx pod that runs with the user ID 101. No need to create the pod
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-nginx
spec:
  securityContext:
    runAsUser: 101
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-nginx
    image: nginx
```


Create the YAML for an nginx pod that has the capabilities "NET_ADMIN", "SYS_TIME" added to its single container
```

apiVersion: v1
kind: Pod
metadata:
  name: security-context-nginx-net-admin
spec:
  containers:
  - name: sec-ctx-nginx-net-admin
    image: nginx
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```

## Resource requests and limits

Create an nginx pod with requests cpu=100m,memory=256Mi and limits cpu=200m,memory=512Mi
```

apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-nginx
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "512Mi"
        cpu: "200m"
      requests:
        memory: "256Mi"
        cpu: "100m"
```

## Limit Ranges

Create a namespace named limitrange with a LimitRange that limits pod memory to a max of 500Mi and min of 100Mi
```

kubectl create ns limitrange
kubectl apply -f limitrange.yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
  namespace: limitrange
spec:
  limits:
  - default: # this section defines default limits
      memory: 500m
    defaultRequest: # this section defines default requests
      memory: 100m
    max: # max and min define the limit range
      memory: "500m"
    min:
      memory: 100m
    type: Container
```

Describe the namespace limitrange
```
kubectl describe ns limitrange
Resource Limits
 Type       Resource  Min   Max   Default Request  Default Limit  Max Limit/Request Ratio
 ----       --------  ---   ---   ---------------  -------------  -----------------------
 Container  memory    100m  500m  100m             500m           -

```


Create an nginx pod that requests 250Mi of memory in the limitrange namespace
```

apiVersion: v1
kind: Pod
metadata:
  name: nginx-250m
  namespace: limitrange
spec:
  containers:
  - name: nginx-250m
    image: nginx
    resources:
      requests:
        memory: "250Mi"

```


## Resource Quotas
Create ResourceQuota in namespace one with hard requests cpu=1, memory=1Gi and hard limits cpu=2, memory=2Gi

```
kubectl create ns one --dry-run=client -o yaml 

apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi

```

Attempt to create a pod with resource requests cpu=2, memory=3Gi and limits cpu=3, memory=4Gi in namespace one

```
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "4Gi"
        cpu: "3"
      requests:
        memory: "3Gi"
        cpu: "2"

% kubectl apply -f one_pod_rq.yaml
Error from server (Forbidden): error when creating "one_pod_rq.yaml": pods "quota-mem-cpu-demo" is forbidden: exceeded quota: mem-cpu-demo, requested: limits.cpu=3,limits.memory=4Gi,requests.cpu=2,requests.memory=3Gi, used: limits.cpu=0,limits.memory=0,requests.cpu=0,requests.memory=0, limited: limits.cpu=2,limits.memory=2Gi,requests.cpu=1,requests.memory=1Gi

```

Create a pod with resource requests cpu=0.5, memory=1Gi and limits cpu=1, memory=2Gi in namespace one


```
apiVersion: v1
kind: Pod
metadata:
  name: quota-mem-cpu-demo
  namespace: one
spec:
  containers:
  - name: quota-mem-cpu-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "2Gi"
        cpu: "1"
      requests:
        memory: "1Gi"
        cpu: "0.5"
```


# Secrets

Create a secret called mysecret with the values password=mypass
```
kubectl create secret generic mysecret --from-literal password=mypass
```


Create a secret called mysecret2 that gets key/value from a file
```
kubectl create secret generic mysecret2 --from-env-file=./username.txt
```

Get the value of mysecret2
```
kubectl get secret mysecret2 -o yaml

data:
  admin: cGFzc3dvcmQ=

echo cGFzc3dvcmQ= | base64 -D
```

Create an nginx pod that mounts the secret mysecret2 in a volume on path /etc/foo
```

apiVersion: v1
kind: Pod
metadata:
  name: nginx-foo
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
      optional: true
```
Delete the pod you just created and mount the variable 'username' from secret mysecret2 onto a new nginx pod in env variable called 'USERNAME'

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-env-secret
spec:
  containers:
  - name: nginx-env-secret
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: mysecret2
          key: admin
```


Create a Secret named 'ext-service-secret' in the namespace 'secret-ops'. Then, provide the key-value pair API_KEY=LmLHbYhsgWZwNifiqaRorH8T as literal.
```
kubectl create ns secret-ops
kubectl create secret generic ext-service-secret --from-literal=API_KEY=LmLHbYhsgWZwNifiqaRorH8T
```

Consuming the Secret. Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops' and consume the Secret as an environment variable. Then, open an interactive shell to the Pod, and print all environment variables.

```
kubectl get secret ext-service-secret -o yaml
apiVersion: v1
data:
  API_KEY: TG1MSGJZaHNnV1p3TmlmaXFhUm9ySDhU
kind: Secret
metadata:
  creationTimestamp: "2024-08-30T07:53:59Z"
  name: ext-service-secret
  namespace: secret-ops
  resourceVersion: "476606"
  uid: 05eb94f2-65cd-44e5-bb7b-3fdb9b8bfe2a
type: Opaque

apiVersion: v1
kind: Pod
metadata:
  name: consumer
  namespace: secret-ops
spec:
  containers:
  - name: consumer
    image: nginx
    env:
    - name: APIKEY
      valueFrom:
        secretKeyRef:
          name: ext-service-secret
          key: API_KEY

 kubectl exec consumer -n secret-ops -it -- /bin/sh
# env
APIKEY=LmLHbYhsgWZwNifiqaRorH8T
```


Create a Secret named 'my-secret' of type 'kubernetes.io/ssh-auth' in the namespace 'secret-ops'. Define a single key named 'ssh-privatekey', and point it to the file 'id_rsa' in this directory
```
kubectl create secret generic my-secret -n secret-ops --from-file=ssh-privatekey=./id_ckad
kubectl get secret my-secret -n secret-ops -o yaml > my-secret-ssh-secret-ops.yaml

apiVersion: v1
data:
  ssh-privatekey: LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFNd0FBQUF0emMyZ3RaVwpReU5UVXhPUUFBQUNBcC9OZ3dCeUJPdHdYL0JvZXMzSGovOUxWL1BJOTB6Nmk0ak9lYlNxek1UZ0FBQUpCUEdJRkJUeGlCClFRQUFBQXR6YzJndFpXUXlOVFV4T1FBQUFDQXAvTmd3QnlCT3R3WC9Cb2VzM0hqLzlMVi9QSTkwejZpNGpPZWJTcXpNVGcKQUFBRUJtKzdBMVlsRTRma0E2bS9ETkdGNzI2ZGI1VzlweHp1S0c2aGFVWmJMUlVTbjgyREFISUU2M0JmOEdoNnpjZVAvMAp0WDg4ajNUUHFMaU01NXRLck14T0FBQUFDMk5oY214dlFGUjVjMjl1QVFJPQotLS0tLUVORCBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0K
kind: Secret
type: kubernetes.io/ssh-auth
metadata:
  name: my-secret
  namespace: secret-ops
type: Opaque
```


Create a Pod named 'consumer' with the image 'nginx' in the namespace 'secret-ops', and consume the Secret as Volume. Mount the Secret as Volume to the path /var/app with read-only access. Open an interactive shell to the Pod, and render the contents of the file.

```
apiVersion: v1
kind: Pod
metadata:
  name: consumer-ssh-pod
  namespace: secret-ops
spec:
  volumes:
    - name: ssh-volume
      secret:
        secretName: my-secret
  containers:
    - name: consumer-ssh-pod
      image: nginx
      volumeMounts:
        - name: ssh-volume
          readOnly: true
          mountPath: "/var/app"


kubectl apply -f nginx_consumer-secret-ops-ssh.yaml 
pod/consumer-ssh-pod created
kubectl get po -n secret-ops
NAME               READY   STATUS    RESTARTS   AGE
consumer           1/1     Running   0          110m
consumer-ssh-pod   1/1     Running   0          7s

kubectl exec consumer-ssh-pod -n secret-ops -it -- /bin/sh
# df
Filesystem     1K-blocks     Used Available Use% Mounted on
overlay         61202244 18555604  39505316  32% /
tmpfs              65536        0     65536   0% /dev
tmpfs            4015504        4   4015500   1% /var/app
/dev/vda1       61202244 18555604  39505316  32% /etc/hosts
shm                65536        0     65536   0% /dev/shm
tmpfs            4015504       12   4015492   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs            2007752        0   2007752   0% /sys/firmware
# cd /var/app
# ls
ssh-privatekey

## ServiceAccounts
