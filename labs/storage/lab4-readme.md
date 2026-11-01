# Lab 4: กู้ข้อมูลจาก PV ที่ Released

Goal: เอา PV เก่า ( จาก lab 3 )กลับมาใช้ และยืนยันว่าข้อมูล retain me ยังอยู่

1) หา PV เก่า
```
kubectl get pv -n storage
```

2) ดู claimRef ที่ขวางอยู่
```
kubectl get pv <OLD_PV> -o yaml | grep -A6 claimRef

eg.
kubectl get pv pvc-683ab63b-9d18-4fcf-a535-9b9e37742391 -o yaml | grep -A6 claimRef
```

ผลลัพธ์
จะเห็นชื่อ lab3-pvc และ uid ของ PVC เก่า
```
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: lab3-pvc
    namespace: storage
    resourceVersion: "383253"
    uid: 683ab63b-9d18-4fcf-a535-9b9e37742391
```

3) ล้าง claimRef เพื่อปล่อย PV ให้ว่าง
```
kubectl patch pv <OLD_PV> -p '{"spec":{"claimRef": null}}'
eg.
kubectl patch pv pvc-683ab63b-9d18-4fcf-a535-9b9e37742391 -p '{"spec":{"claimRef": null}}'

kubectl get pv <OLD_PV>      # STATUS ควรเปลี่ยนเป็น Available
```
ผลคือ PV status จาก released เป็น available


4) สร้าง PVC ใหม่ที่ชี้ PV นั้นตรงๆ และ Pod สำหรับตรวจข้อมูล
lab4.yaml
```
kubectl apply -f lab4.yaml
kubectl get pvc,pv -n storage
kubectl exec -n storage lab4 -- cat /data/note.txt
```
ผลลัพธ์
- ได้ PVC และ pod ใหม่่
- ตัว PV เก่า เปลี่ยน status เป็น bound เข้ากับ PVC ใหม่ที่ถูกสร้าง
- และเมื่อ cat ไฟล์ note ของ pod ที่ mount volume ตัว PV เก่า ออกมา ก็จะได้ค่าเหมิือนเดิมก่อนหน้าที่เคยสร้างใน lab3

---

5) เก็บกวาด (สำคัญ เพราะ keep-data เป็น Retain ลบ PVC แล้วดิสก์จะไม่หายเอง)
```
kubectl delete pod lab4 lab3 -n storage
kubectl delete pvc lab4-pvc lab3-pvc -n storage
kubectl get pv -n storage                     # PV ที่ค้างเป็น 

Released
kubectl delete pv <ชื่อ PV ที่ค้าง>
eg.
kubectl delete pv persistentvolume/pvc-612747aa-b38b-4b81-a92f-87202e63c299 -n storage

kubectl delete pv persistentvolume/pvc-683ab63b-9d18-4fcf-a535-9b9e37742391 -n storage
```
สุดท้าย lab จะมาทำการ clean PVC, PV ออกทั้งหมด
- มาเริ่มด้วยการลบ pod กันออกก่อน เพืิ่อไม่ให้มี resoruce อะไรไปผูกกับ PVC
- จากนั้นเมื่อลบ pvc จะเหลือ pv อยู่เพราะ มี reclaim_policy เป็น retain (จึงทำให้ pv ไม่ถูกลบตาม pvc ไป)
- ในการลบ pv จะต้องลบมือ โดยลบ pv เป็นตัวๆ ไป

---

### คำถามให้คิด
(ลองตอบ)
1. ทำไมใน PVC เราต้องใส่ volumeName ด้วย ถ้าไม่ใส่ จะเกิดอะไรขึ้น?
ANS: ก็ถ้าไม่ใส่ตอนสร้างตัว pvc ก็จะสั่งให้ storage class สร้าง pv ขึ้นมาใหม่ แต่ถ้าเราใส่เสมือนเป็นการบังครับให้ pvc ชี้ไปที่ pv ที่มีอยู่

2. ข้อ 4 ผลของ cat เป็นอะไร และการกู้นี้ต่างจาก Lab 2 อย่างไร?
ANS: ก็เป็นค่า "retain me", ต่างจาก lab 2 คือใน lab ที่ 2 ตัว pvc ใช้ storage class ที่เป็น default คือมี recliam_policy เป็น delete นั่นทำให้เมื่อเราลบ PVC ส่งผลให้ PV ถูกลบตามไปด้วย. ซึ่งแตกต่างจาก lab ที่ pvc ใช้ storage class ชื่อ keep-data ซึ่งมี reclaim_policy เป็น retain ซึ่งส่งผลให้เมื่ิอเราลบ PVC แต่ตัว pv ไม่ถูกลบตาม

(เฉลย)
ตอบถูกทั้งสองข้อเลยครับ และข้อ 2 อธิบายสาเหตุได้ครบ ทั้ง default class (Delete) กับ keep-data (Retain) แปลว่าคุณเข้าใจ reclaim policy แล้ว

มีรายละเอียดเล็กน้อยให้เก็บไว้ในหัว:

ข้อ 1: เสริมอีกนิดว่าถ้าใส่ volumeName แล้ว PV นั้นไม่ตรงเงื่อนไข (เช่น ขนาดเล็กกว่าที่ขอ, access mode ไม่ตรง, storageClass ไม่ตรง หรือ PV ผูกกับ PVC อื่นอยู่) PVC จะค้างเป็น Pending ไม่ไปสร้าง PV ใหม่ให้ ซึ่งเป็นพฤติกรรมที่ตั้งใจ เพราะคุณสั่งให้ใช้ตัวนี้โดยเฉพาะ
ข้อ 2: ความต่างที่แท้จริงคือ Lab 2 เป็นการ "ลบ Pod แต่ PVC อยู่" ส่วน Lab 4 เป็นการ "ลบ PVC ไปแล้ว แล้วกู้ข้อมูลจาก PV ที่ Retain ไว้" ซึ่งเป็นสถานการณ์ที่หนักกว่ามาก

