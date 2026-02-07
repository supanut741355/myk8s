### Create pod

```
syntax
$ kubectl run [POD_NAME] --image [APP_IMAGE_NAME]:[TAG]

eg.

$ kubectl run myNginx --image nginx
```


### Check status of pod
```
$ kubectl get pods
```

Can add flag -o wide to see more detail
```
$ kubectl get pods -o wide
```
detail:
- Node that pod exist
- ip of pod




### Get description of pod
```
syntax
$ kubectl describe pod [POD_NAME]

eg.
$ kubectl describe pod myNginx
```