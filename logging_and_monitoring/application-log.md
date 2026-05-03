# Application logging



### Docker
1. docker run 
```
$ docker run -d <ima_name>
```
2. view docker logs
```
$ kubectl logs -f <container_id>
```

### K8s
1. create the pod 
```
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod

spec:
  containers:
    - name: event-simulator
      image: kodekloude/event-simulator
```

1.1
```
$ kubectl apply -f event-simu.yaml
```


2. View the log 
```
$ kubectl logs -f pod event-simulator-pod
```


In the case has multi container in pod
```
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod

spec:
  containers:
    - name: event-simulator
      image: kodekloude/event-simulator
    - name: img-processor
      image: kodekloude/img-processor
```

view log with
```
$ kubectl logs -f <type> <metadata.name> <spec.containers.name>

eg.
$ kubectl logs -f pod pod event-simulator-pod vent-simulator
```