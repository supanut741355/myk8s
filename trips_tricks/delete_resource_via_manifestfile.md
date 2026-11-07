สมมุติเรามี manifest file คือ aaa.yaml 

และเราได้ทำการ apply -f aaa.yaml ไป

ต่อมาต้องการจะ clean reinstall ทำได้โดย

```
kubectl delete -f aaa.yaml -n <namespace>
```