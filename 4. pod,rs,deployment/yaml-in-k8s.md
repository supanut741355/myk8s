YAML in k8s


### Top level of root properties (4 properties)
```
apiVersion:
kind:
metadata:





spec:
```




### kind = refer to type of object to create
```
  kind: pod
```

- pod
- deployment
- replicaset
- service

### metadata = data about the object 
```
  metadata:
    name: myyoapp-pod
    labels:
      app: myyoapp
      type: front-end
```

### spec = properies of application that need to run in pod
```
  spec:
    containers:
      - name: nginx-container
        image: nginx

```


Full manifest (pod-definition.yaml)
```
apiVersion: v1
kind: pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end

spec:
  containers:
    - name: nginx-container
      image: nginx
```


---

# Create

Run create by k8s cli
```
$ kubectl create -f [PATH_TO_FILE]

eg.

$ kubectl create -f pod-definition.yaml

or

$ kubectl apply -f [PATH_TO_FILE]

eg.

$ kubectl apply -f pod-definition.yaml

```

# Get pod
```
$ kubectl get pods
```

# Get detail
```
$ kubetl describe pod [metadata.name]

eg.

$ kubectl describe pod myapp-pod
```