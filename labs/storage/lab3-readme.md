# Lab 3: Reclaim Policy (Delete vs Retain)

Goal: เห็นความต่างระหว่าง Delete กับ Retain และรู้วิธีกู้ข้อมูลเมื่อใช้ Retain



1) สร้าง StorageClass ใหม่ที่ใช้ policy เป็น "Retain"

เริ่มด้วย ตรวจ provisioner ของ default ก่อน เพราะต้องใช้ตัวเดียวกัน
```
kubectl get storageclass -o wide
```

จากไฟล์ lab3.yaml -> ดูคอลัมน์ PROVISIONER (minikube มักเป็น k8s.io/minikube-hostpath, kind มักเป็น rancher.io/local-path) แล้วสร้างไฟล์ lab3.yaml โดยแทนค่า <PROVISIONER> ด้วยของคุณ
= กรณีนี้ เราใช้ k8s.io/minikube-hostpath


2) รัน เขียนข้อมูล แล้วลบ PVC

```
//Part สร้าง resource
kubectl apply -f lab3.yaml
kubectl get pvc,pv -n storage                              # รอจนเป็น Bound
kubectl exec lab3 -n storage -- sh -c "echo 'retain me' > /data/note.txt"
kubectl exec lab3 -n storage -- cat /data/note.txt 

//Part เล่นกับ resoruce
kubectl delete pod lab3 -n storage
kubectl delete pvc lab3-pvc -n storage
kubectl get pv -n storage

```

ผลคือเราได้ 

### (part สร้าง resource)
- storage class อันใหม่ ที่มี reclaimpolicy เป็น Retain 
- PVC ที่มี mode เป็น RWO และ match กับ storage class
- PV ที่มี mode เป็น RWO, reclaim_policy เป็น Retain และตัวที่ claim ไปคือ PVC และ storage class ข้างบน และ มี status เป็น bound
- Pod ที่ mount volume ไปยัง PVC


### Part เล่นกับ resoruce
!! ข้อสำคัญคือ ถ้า PVC มี pod bound อยู่ และถ้าสั่งลบ PVC จะติด status: terminating ค้าง ละ เพราะ PVC จะมี Finalizer เช่น kubernetes.io/pvc-protection ป้องกันการลบขณะที่ยังมี Pod ใช้งาน -> ดังนั้นเราจะต้องลบ resource ที่ใช้งานตัว pvc นี้ให้หมดก่อน ถึงจะสามารถลบ pvc ได้

//----- แวะ -----
Finalizer ใน Kubernetes คือกลไกที่ใช้ป้องกันไม่ให้ Resource ถูกลบทันที จนกว่าจะทำงานบางอย่างให้เสร็จก่อน -> พูดง่าย ๆ คือ เป็นเหมือน Deletion Protection / Cleanup Hook ที่ Kubernetes ใช้ควบคุมขั้นตอนการลบ Resource ครับ
จากรูป finalizer_flow_to_product_delete.png

1.API Server รับคำสั่ง Delete
2.ถ้ามี Finalizer จะกำหนด deletionTimestamp
3.Resource จะเข้าสู่สถานะ Terminating
4.Controller ที่รับผิดชอบจะทำ Cleanup
5.เมื่อ Cleanup เสร็จ จะลบ Finalizer ออกจาก Resource
6.Kubernetes จึงลบ Resource ออกจาก API ได้จริง
//---------------

เมื่อลบ PVC ออกไปแล้ว ตัว PV จะไม่หาย แต่่ status จะถูกเปลี่ยนจาก bound -> release


### Part ต่อเนื่อง
```
kubectl apply -f lab3.yaml -n storage
kubectl get pvc,pv -n storage
```

ลองดูว่าถ้าสร้าง PVC ใหม่ มันจะเชื่่อมกับ PV เดิม หรือมันจะสร้าง PV ใหม่
ผลคือ PVC ถูกสร้าง และ ได้ PV ตัวใหม่ -> PVC ใหม่ควรได้ PV ใหม่ ไม่ใช่ตัวเก่า เพราะ PV ที่ Released ยังมีร่องรอยของ PVC เดิมผูกอยู่ ทำให้ถูกใช้ซ้ำอัตโนมัติไม่ได้

---
คำถามให้คิด
(ลองตอบ)
1. ในข้อ 2 สถานะของ PV หลังลบ PVC เป็นอะไร และข้อมูลยังอยู่ไหม?
ANS: เป็น release, คิดว่าข้อมูลยังอยู่นะ แต่ไม่รู้ว่ามีวิธีเช็คยังไง

2. ทำไม PVC ใหม่ในข้อ 4 ถึงไม่ได้ PV เดิมกลับมา?
ANS: คิดว่าตอนที่สร้าง PVC ใหม่ มันไปบอกให้ storageclass สร้าง PV ใหม่ ให้ด้วย ถูกไหมนะ

หรือมีวิธีที่ทำให้ PVC ใหม่ สามารถใช้ PV ตัวที่ยังคงอยู่ได้ไหมนะ

3. ถ้าเป็นฐานข้อมูล production คุณจะเลือก Delete หรือ Retain เพราะอะไร?
ANS: คิดว่าต้องเป็น retain นะ เพราะใช้ในกรณีเผลอลบ PVC ป่ะ

(เฉลย)
ข้อ 1: ถูกครับ ข้อมูลยังอยู่ เพราะ Retain สั่งให้ k8s "ไม่ลบดิสก์ให้" วิธีเช็กที่ชัวร์ที่สุดคือเอา PV นั้นไปต่อกับ Pod แล้วเปิดไฟล์ดู ซึ่งเราจะทำใน Lab 4

ข้อ 2: ถูกครึ่งหนึ่งครับ ส่วนที่คุณเห็นถูกคือ PVC ใหม่ทำให้ StorageClass สร้าง PV ใหม่ แต่สาเหตุที่ไม่ได้ของเดิมคือ PV เก่ามี claimRef ชี้ไปที่ PVC ตัวเก่า (จำทั้งชื่อและ UID) สถานะเลยเป็น Released ไม่ใช่ Available k8s จึงไม่เอาไปให้ใครอัตโนมัติ เผื่อมีข้อมูลเก่าของคนอื่นค้างอยู่

และคำถามของคุณว่า "มีวิธีให้ PVC ใหม่ใช้ PV เดิมไหม" มีครับ นั่นคือ Lab 4

ข้อ 3: Retain ถูกต้องครับ ที่คุณเดาคือเหตุผลหลัก (เผื่อเผลอลบ PVC) มีข้อแลกเปลี่ยนสองข้อที่ควรรู้ไว้

ดิสก์ที่ไม่มีใครใช้จะค้างอยู่ ต้องมีคนตามลบเองเพื่อไม่ให้เสียเงินเปล่า
Retain ไม่ใช่ backup ถ้าดิสก์เสียหรือข้อมูลถูกเขียนทับ ก็ยังหายอยู่ดี production จริงต้องมี snapshot หรือ backup แยกด้วย

