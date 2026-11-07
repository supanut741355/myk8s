# Lab 5: Access Modes

Goal: เข้าใจว่า volume เดียวกันให้กี่ Pod ใช้พร้อมกันได้ และทำไม ReadWriteOnce ถึงเป็นข้อจำกัดที่เจอบ่อย

Mode	                | ความหมาย
__________________________________________________________
ReadWriteOnce (RWO)	    | อ่าน/เขียนได้ จาก node เดียว ณ เวลาหนึ่ง
ReadOnlyMany (ROX)	    | อ่านอย่างเดียว จากหลาย node
ReadWriteMany (RWX)	    | อ่าน/เขียนได้ จากหลาย node
ReadWriteOncePod (RWOP)	| อ่าน/เขียนได้ จาก Pod เดียว เท่านั้น

!!จุดที่หลายคนเข้าใจผิดคือ RWO จำกัดที่ระดับ node ไม่ใช่ Pod ดังนั้นหลาย Pod บน node เดียวกันยังใช้ PVC เดียวกันได้ **แต่คนละ node ไม่ได้**


---
Create resource
```
kubectl apply -f lab5.yaml -n storage
kubectl get pods lab5-a lab5-b -n storage
kubectl describe pod lab5-b | tail -15 -n storage
```
Result
- มีการสร้าง PVC ซึ่งมี policy เป็น "ReadWriteOncePod" และ PV 
- มี 2 pod คือ lab5-a, lab5-b ซึ่งทั้ง 2 ตัว mount volume ไปที่ PVC เดียวกัน
เห็น Pod หนึ่งตัว Running อีกตัวค้างที่ Pending ให้ดูข้อความในส่วน Events ว่าเขียนว่าอะไร ด้วย warning error: 
    Warning  FailedScheduling  72s (x2 over 72s)  default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. not found

---
มาเปลี่ยน mode
```
//ลบของเดิมก่อน แล้วแก้ accessModes เป็น ["ReadWriteOnce"] รันใหม่
kubectl delete -f lab5.yaml
# แก้ไฟล์: ReadWriteOncePod -> ReadWriteOnce
kubectl apply -f lab5.yaml
kubectl get pods -o wide
```
ผลลัพธ์:
- ทุก pod สามารถ running ได้

บน minikube/kind ที่มี node เดียว คราวนี้ทั้งสอง Pod น่าจะ Running ได้ ลองดูคอลัมน์ NODE ว่าอยู่ node เดียวกันหรือไม่
```
lab5-a   1/1     Running   0          31s   10.244.0.19   minikube   <none>           <none>
lab5-b   1/1     Running   0          31s   10.244.0.20   minikube   <none>           <none>
```

---
Clean lab
```
kubectl delete -f lab5.yaml
```