# What is replica
scenario: we have a single pod run a application -> pod crash or fail => user no longer able to access app. To Prevent user from losing access -> need more than one pod running at the same time.


# Why need replication controller
The replication controller help us run multi ple single pod in kubecluster = provide High Availability. 

So if we have a single pod, the replication controller can help by automatically bring up the new pod, when existing one fail.


# Additional reason of must having replication controller
is create a multi pod to share a load (be a LB and scaling)

replication controller can manage pod across node. 

# Replicaton controller & Replicaset
Both is the same purpose
- Replicatio controller is old tech is replaced by ReplicaSet




# Diff major RC and RS
is selector

```
selector
  matchLabels:
    xxxx: yyyyy
```

match label selector is match the label spec under the label on pods


# Labels and Selector
(TL;DR: just attach a tag to pod)
Why we label pod and object?

simple scenario: We deploy 3 instance of app, we create replicaset to ensure we have 3 active pod at any time. => can use to monitor exist pod 


role of replicaSet is to monitor the pod  -> if any of them fail -> deploy new one.


Q: How replicaset know what pod to monitor ?? (100+ pod in cluster)
A: the Labels come to handy


pod.yaml
```
metadata:
  name: myapp-pod
  labels:
    type: front-end
```


replicaset.yaml
```
metadata:
  matchLabels:
    tier: frontend
```

this way the replicaset know what pod to monitor


---

The concept Label and Selector is used in many other place in k8s

# Scale the replicaSet

- method1: Edit the number of replicaset in manifest file
- method2: use cli
```
$ kubectl scale --replicas=6 -f [MANIFEST_FILE_NAME].yaml
```

- method3: use cli
```
$ kubectl scale --replicas=6 [TYPE] [NAME]

eg.
$ kubectl scale --replicas=6 replicaset myapp-replica.yaml

```