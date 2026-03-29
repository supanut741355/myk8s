# DaemonSet

ReplicaSet: responsibility to manage and ensure copy of app pod in node.


DaemonSet: reponsibility to manage and ensure copy of app pod across (distribute) to every node.
1 pod per node.

Whenever number of node is up, the daemonSet will deploy the app pod to the new node and when number of node is down daemonSet will delete the app pod.



### Use-cases 
1. Monitoring solution
2. Log viwer
3. Kube-proxy is a sample of daemonSet
4. Networking solution (calico)
