PV, PVC, Storage class
ตัวละคร	| เปรียบเทียบ | หน้าที่
PV (PersistentVolume) |	ดิสก์จริงที่ถูกจัดสรรไว้ | ตัวพื้นที่เก็บข้อมูลจริง
PVC (PersistentVolumeClaim) | ใบขอใช้พื้นที่ | Pod ขอ "อยากได้ 1Gi" ผ่านใบนี้
StorageClass | ประเภท/โรงงานผลิตดิสก์ | บอกว่าจะสร้าง PV แบบไหน และสร้างให้อัตโนมัติ

---

# Lab 2: ข้อมูลรอดแม้ลบ Pod

## ดู storage class ที่มีในเครื่อง
```
$ kubectl get storageclass
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lab2-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: lab2
spec:
  containers:
    - name: app
      image: busybox
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: mydata
          mountPath: /data
  volumes:
    - name: mydata
      persistentVolumeClaim:
        claimName: lab2-pvc
```

---

```
kubectl apply -f lab2.yaml -n storage
kubectl get pvc -n storage         # STATUS ควรเป็น Bound (บาง cluster เป็น Pending จนกว่า Pod จะถูกสร้าง ซึ่งปกติ)
kubectl get pv -n storage        # เห็น PV ที่ถูกสร้างอัตโนมัติ ชื่อขึ้นต้น pvc-...
```

```
kubectl exec lab2 -n storage -- sh -c "echo 'i survive' > /data/note.txt" 
kubectl delete pod lab2 -n storage
kubectl apply -f lab2.yaml -n storage         # PVC ไม่ถูกแตะ สร้าง Pod ใหม่
kubectl exec lab2 -n storage -- cat /data/note.txt 
```

---

คำถามให้คิด
1. ข้อ 4 ผลที่ได้คืออะไร และต่างจาก Lab 1 อย่างไร?
ANS: ที่ต่างจาก lab1 คือ ไฟล์ที่ถูกสร้างจาก pod จะอยู่เก็บใน volumn จึงทำให้เมื่อลบ pod ไฟล์จึงไม่หาย -> ข้อมูลอยู่ใน volume ที่แยกอายุจาก Pod

2. เราไม่ได้สร้าง PV เอง แล้ว PV โผล่มาได้อย่างไร? (ใบ้: ดูหัวข้อ StorageClass ในตาราง)
ANS: PVC แค่ "ขอ" ส่วนคนที่สร้าง PV ให้จริงคือ StorageClass (ผ่านตัว provisioner ที่อยู่เบื้องหลัง) ลำดับคือ

1)เราสร้าง PVC (ไม่ได้ระบุ StorageClass จึงใช้ตัว default)
2)StorageClass default เห็นคำขอ แล้วสั่ง provisioner สร้างดิสก์
3)ได้ PV ใหม่ แล้ว k8s ผูก PV กับ PVC ให้ (สถานะ Bound)
เรียกว่า dynamic provisioning ครับ

---

สิ่งที่น่าลองต่อ ก่อนไป Lab 3
ลองลบ PVC แล้วดูว่า PV เกิดอะไรขึ้น

```
kubectl delete pod lab2 -n storage
kubectl delete pvc lab2-pvc -n storage
kubectl get pv -n storage
```
ถ้า kubectl get pv ไม่เหลือ PV นั้น ให้ลองเดาว่าทำไม แล้วดูคอลัมน์ RECLAIM POLICY ใน kubectl get storageclass -o wide ประกอบ จะนำไปสู่เรื่อง reclaim policy (Delete vs Retain) ใน Lab 3 ครับ

PV หายเพราะ reclaim policy เป็น Delete หมายความว่าเมื่อ PVC ถูกลบ k8s จะลบ PV และดิสก์จริงที่อยู่เบื้องหลังทิ้งด้วย ข้อมูลหายถาวร

นี่คือจุดที่คนมือใหม่พลาดบ่อย: ลบ PVC เพลินๆ ข้อมูลหายหมด