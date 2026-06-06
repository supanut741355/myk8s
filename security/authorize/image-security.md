# Secure image

image: nginx
image: docker/libraby/nginx

```
apiVersion: v1
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```


### Privaty registry

```
$ docker login [private-registry.io]

$ docker run private-registry.io.apps/internal-app

$ kubectl create sercret docker-registry regcred \
  --docker-servicer= [private-registry.io]\
  --docker-userbane= [registry-user]\
  --docker-password= [registry-password]\
  --docker-email= registry-user@org.com
```

```
apiVersion: v1
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: private-registry.io/apps/internal-app
  imagePullSecrets:
    - name: regcred
```


