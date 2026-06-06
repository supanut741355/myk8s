2 type of account in K8s

1. user account -> used by human. eg. admin, developer
2. service account -> used by machine eg.  application (prometheus, jenkins)


eg. เราทำ app k8s dashboard ขึ้นมาตัวนึง ซึ่ง data ที่จะแสดงได้มาจาก api ที่ส่งไปที่ kube-api => app ไป query ของใน k8s ซึ่งจะต้องการการ authen -> นั่นคือหน้าทีหลักของ service account


# service account ใช้ในการ Authen

คิดซะว่า service account คือ token / password ที่ใช้ในการ authen

--- 
โดย default แล้ว k8s จะมี service account 1 ตัวชื่อ default
```
$ kubectl get serviceaccount

$ kubectl describe serviceaccount [SA_NAME]
```


ตัว SA จะถูก mount ไปที่ volume ของ pod  ที่
```
/var/run/secrets/kubernetes.io/serviceaccount
```

### Create service account
```
$ kubectl create serviceaccount [dashboard-sa]
```

or
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
```

### Associate SA to pod
```
apiVersion: v1
metadata:
  name: my-kube-dashboard
space:
  containers:
    - name: my-k8s-dashboard
      image: my-k8s-dashboard
  serviceAccountName: dashboard-sa
```

# Create token
k8s will create a token with expiry date and automaticakky mount to pod

```
$ kubectl create token dashborad-sa
``