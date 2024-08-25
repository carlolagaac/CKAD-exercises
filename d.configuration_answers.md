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




