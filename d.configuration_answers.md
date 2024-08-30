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
kubectl create secret generic mysecret2 --from-env-file=./mysecret2.txt
```

Get the value of mysecret2
```
kubectl get secret mysecret2 -o yaml

  password2: YWJjMTIz
  password3: ZGVmNDU2

echo YWJjMTIz | base64 -D
```

