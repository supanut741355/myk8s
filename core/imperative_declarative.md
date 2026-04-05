IaC

### Imperative 
: provision infra by set of instruction step by step
eg.
1. Provision VM name web-server
2. Install Nginx
3. Edit config nginx file port 8080
4. Edit config file web path /var/www/nginx
5. Reload web page 
6. Start Nginx server

### Declarative =>
:declear the requirements

```
vm_name: web-server
package: nginx
port: 8080
path: /var/www/nginx
code: git repo
```


--------

In K8s

### Imperative
```
$ kubectl run --image=nginx nginx

$ kubectl create deployment --image=nginx nginx

$ kubectl expose deployment nginx --port 80

$ kubectl edit deployment nginx 

$ kubectl scale deployment nginx --replicas=5

$ kubectl set image deployment nginx nginx=nginx:1.18
```


### Declarative

$ kubectl apply -f nginx.yaml


nginx.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  label:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx

```


