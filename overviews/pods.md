Assumtion we have a application Docker image and k8s

K8s not deploy container directly on worker node -> the contianer are encapsulated in pod

pod = single instance of application

To scale the application for users access -> we need add pods to share the load. by create new pod with same application in node

But if user further increase and node has no sufficient capacity -> we need to deploy the pod on a new node in cluster (add node to cluster to expand the cluster physical capacity)

scale up = create new pods.
scale down = delete existing pods.

---

Multi container pod (rare case)

the signle pod can have multiple container

sometime scenario: it has a helper container that live alongside with main application (sidecar). When pod create the application and helper (sidecar) will be create, and when it die -> helper also dies.

2 container in pods can communicate each other as localhost = share same network  and share storage space.


---

Create pod in kubernetes

deploy docker container by creating a pod
```
$ kubectl run nginx
```
^ It will create pod and deploy a instance of nginx docker image.

add flag image name
```
$ kubectl run nginx --image nginx
```

the nginx image will download from Docker hub repository.


To see the list of pod available 
```
$ kubectl get pod
``