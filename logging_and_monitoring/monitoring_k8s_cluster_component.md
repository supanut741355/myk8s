# Monitor

- Resource comsumption
- Node-level metric
  - number of node 
  - number of node are healthy
  - metric cpu, memory, network and disk

- Pod-level metric
  - resource/performance of each pod


k8s has not built-in solution -> open-source solution is
- Metric server
- Prometheus
- ElasticStack



### Metric server
- Retrieve metric from each k8s's node and pod -> aggregate and store in memory
- Only work at in-memory (ram) -> can store metric on disk = cant see history


### How metric generated for the pod on node ?
- k8s run agent on each node (aka kubelet) -> responsible for reciving the instruction from kube-api-server.

- kubelet cotaine sub-component -> **cAdvisor**

- cAdvisor is responsible for retrieving performance metric from pod  and exposing them to kubelet api to make metric available to metric server.


eg.
using minikube:
1. nable Metric server
```
$ minikube addons enable metric-server
```

2. clone metric server deployment file
```
$ git clone https://github.com/kubernetes-sigs/metrics-server.git
```

3. deploy the require componenet:
- set of pod
- service
- role
```
$ kubectl create -f deploy/1.8+/
```


### Using

- View node's CPU and memory comsumption of each node.
```
$ kubectl top node
```

- View pod's CPU and memory comsumption of each pod.
```
$ kubectl top pod
```