# RBAC

by create a role component

eg. We need RBAC role call "developer" that can:
- View pods
- Create pods
- Delete pods
- create configMap


### // STEP 1: Create rule (kind: Role)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]

  - apiGroups: [""]
    resources: ["configMap"]
    verbs: ["create"]

```


each rule has 3 field
```
apiGroups:
resourecs:
verbs
```

### // STEP2: Link user to rule (kind: RoleBinding)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```


## View RBAC
```
$ kubectl get roles


$ kubectl get rolebindings


$ kubectl description role [ROLE_NAME]

$ kubectl description rolebinding [ROLE_BINDING_NAME]
```