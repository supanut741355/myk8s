# Lab 7: ต่อ PVC เดิมกลับเข้า StatefulSet

เป้าหมาย: พิสูจน์ว่า StatefulSet ใหม่ใช้ข้อมูลเก่าได้ ตราบใดที่ PVC ยังอยู่

ตอนนี้ PVC data-web-0..2 ควรยังอยู่ (ถ้าคุณยังไม่ได้ลบตามข้อ 5 ของ Lab 6) เช็กก่อน


1) เขียนข้อมูลเพิ่มแล้ว scale down
```
kubectl apply -f lab6.yaml
kubectl exec web-2 -- sh -c "echo 'before scale down' >> /data/who.txt"
kubectl scale statefulset web --replicas=1
kubectl get pods
kubectl get pvc
```
ผล
- เพิ่มข้อมูลที่ pod web-2
- ผลคือตอน scale ลง -> ตัว pod จะเหลือ 1
- ตัว PVC ยังอยู๋

2) scale กลับขึ้น แล้วอ่านข้อมูล
kubectl scale statefulset web --replicas=3
kubectl get pods -w          # รอจน web-2 Running
kubectl exec web-2 -- cat /data/who.txt

ผล
- ข้อมูลทีเพิ่มเข้าไป ยังอยู่

3) เก็บกวาด
```
kubectl delete -f lab6.yaml
kubectl delete pvc data-web-0 data-web-1 data-web-2
```

---
คำถามให้คิด
ข้อ 1 หลัง scale down เหลือ 1 ตัว PVC ของ web-1 และ web-2 ยังอยู่ไหม?
ข้อ 2 web-2 ตัวใหม่เห็นบรรทัด before scale down ไหม แปลว่า k8s จับคู่ Pod กับ PVC จากอะไร?
ถ้าเราตั้ง persistentVolumeClaimRetentionPolicy.whenScaled: Delete ผลของข้อ 1 จะเปลี่ยนอย่างไร และสถานการณ์ไหนที่คุณจะเลือกแบบนั้น?

// ลองตอบ
1. ยังอยู่

2. เห็น, คิดว่าจับจาก mount volume

3. คิดว่า PVC จะถูกลบ / recreate ถ้ามีการ scale ขึ้นหรือลง. คิดกรณีไม่ออก

// เฉลย
ข้อ 1: ถูกต้อง

PVC ของ web-1 และ web-2 ยังอยู่ เพราะค่าเริ่มต้นของ whenScaled คือ Retain

ข้อ 2: เห็นถูกแล้ว แต่ขอปรับเหตุผล

คำว่า "จับจาก mount volume" ยังไม่ตรงนัก การจับคู่เกิดจาก ชื่อ ครับ StatefulSet controller ตั้งชื่อ PVC ตามสูตร <ชื่อ template>-<ชื่อ StatefulSet>-<ลำดับ> แล้วตอนสร้าง Pod ก็ใส่ claimName ชี้ชื่อนั้นให้ ดังนั้น Pod ชื่อ web-2 จึงเจอ PVC data-web-2 เสมอ และถ้า PVC ชื่อนี้มีอยู่แล้ว (ค้างจากรอบก่อน) ก็จะถูกนำกลับมาใช้ แทนที่จะสร้างใหม่

นี่คือเหตุผลที่ Pod ใน StatefulSet มี ลำดับและชื่อคงที่ ตัวตนของ Pod เป็นตัวกำหนดว่าจะได้ข้อมูลก้อนไหน

ข้อ 3: ทิศทางถูก ขอเรียบเรียงใหม่

พฤติกรรมของ whenScaled: Delete คือ

scale ลง: PVC ของ replica ที่ถูกตัดออก (เช่น data-web-1, data-web-2) จะถูกลบ
scale ขึ้นอีกครั้ง: PVC ถูกสร้างใหม่ ว่างเปล่า ข้อมูลเก่าไม่กลับมา

ที่คุณเดาว่า "ถูกลบ/recreate" จึงถูก แต่แยกให้ชัดว่าลบตอนลง สร้างใหม่ตอนขึ้น และ PV ข้างหลังจะหายตามหรือไม่ยังขึ้นกับ reclaim policy ของ StorageClass อีกชั้นหนึ่ง

กรณีที่ควรเลือก Delete (ที่คุณคิดไม่ออก)

replica ที่ข้อมูลสร้างใหม่ได้จากตัวอื่น เช่น read replica ที่ sync จาก primary ได้ หรือ node ของ cache/search ที่ rebuild index ได้
ต้องการประหยัดค่าดิสก์: scale ลงแล้วไม่อยากจ่ายค่าดิสก์ที่ไม่มีใครใช้
workload ชั่วคราว เช่น ชุดทดสอบที่ scale ขึ้นลงบ่อย

กรณีที่ควรใช้ Retain

ข้อมูลหลักที่แทนไม่ได้ เช่น primary ของฐานข้อมูล หรือ replica ที่การ resync ใช้เวลานานมากหรือเสี่ยง

หลักคิดคือ ถามว่า "ถ้าดิสก์ก้อนนี้หาย จะสร้างข้อมูลคืนได้ง่ายไหม" ถ้าได้ง่ายเลือก Delete ถ้าไม่ได้เลือก Retain



---

