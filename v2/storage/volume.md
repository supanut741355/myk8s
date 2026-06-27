## Volume in Docker
- volume will attach (mount) to continer



## Volume in K8s
- volume will attach (mount) to pod

<Internal volume in node>
```
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out"]
    volumeMounths:
      - mountPath: /opt
        name: data-volume

  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```



<Internal volume with volume driver plugin eg. aws ebs>
```
  volumes:
    - name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4

```
