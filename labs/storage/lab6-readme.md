# Lab 6: StatefulSet และ volumeClaimTemplates

เป้าหมาย: เห็นว่าแต่ละ replica ได้ PVC ของตัวเอง และข้อมูลตามกลับมาเมื่อ Pod ถูกสร้างใหม่

### สร้าง resource
```
kubectl apply -f lab6.yaml
kubectl get pods -w          # กด Ctrl+C เมื่อครบ 3 ตัว
kubectl get pvc
kubectl exec web-1 -- cat /data/who.txt
```

ผลลัพธ์
1. Service แบบ clusterIP: None (headless) เป็นข้อกำหนดของ StatefulSet ไว้ให้แต่ละ Pod มีชื่อเครือข่ายคงที่ ตอนนี้ยังไม่ต้องสนใจรายละเอียด

2. ตัว statefulset ซึ่งมี 3 pod (สังเกตชื่อ Pod (ไม่ใช่ชื่อสุ่มแบบ Deployment), ลำดับการสร้าง และชื่อ PVC)

3. ได้ตัว PVC 3 ตัว ผูกกับ pod แต่ละตัว



### Interact with Pod
```
kubectl delete pod web-1
kubectl get pods              # รอให้ web-1 กลับมา Running
kubectl exec web-1 -- cat /data/who.txt
```

ผล
- เมื่อลบ pod web-1 รอสักแปป มันจะกลับขึ้นมารันใหม่ ซึ่งจะมีข้อความเพิ่มขึ้นมาอีกบรรทัด



###  Interact with Stateful - ลบ StatefulSet แล้วดู PVC
```
kubectl delete -f lab6.yaml
kubectl get pvc
```


### Clean up
```
kubectl delete pvc data-web-0 data-web-1 data-web-2
```


---
คำถามให้คิด
1. ในข้อ 2 PVC มีกี่ตัว ชื่ออะไร และต่างจาก Deployment ที่ใช้ PVC ตัวเดียวร่วมกันอย่างไร?
2. ในข้อ 3 web-1 ตัวใหม่ใช้ PVC ตัวไหน และเห็นอะไรในไฟล์?
3. ในข้อ 4 หลังลบ StatefulSet PVC ยังอยู่ไหม คุณคิดว่าทำไม k8s ถึงออกแบบแบบนี้?

//ลองตอบ
1. มี 3 ตัว ชื่ีอ data ตามด้วยชื่อ ต่างจาก pvc ของ deployment ที่ชื่อจะเป็น hash
2. ใช้ PVC data-web-1
3. ยังอยู่ คิดว่า k8s ไว้ใช้กันถ้าเผลอลบ statuful ตัวข้อมูลที่ pvc, pv จะยังไม่หาย

//เฉลย
ข้อ 1: ถูกส่วนแรก ส่วนหลังต้องแก้

ส่วนที่ถูกคือ PVC มี 3 ตัว ชื่อ data-web-0, data-web-1, data-web-2 ชื่อมาจาก <ชื่อ template>-<ชื่อ StatefulSet>-<ลำดับ> และแต่ละ replica ได้ PVC ของตัวเอง

ส่วนที่ต้องแก้คือเรื่อง hash Deployment ไม่ได้สร้าง PVC ให้เลยครับ hash ที่เห็นเป็นต่อท้ายชื่อ Pod (เช่น myapp-7d9f8b-xk2lp) ไม่ใช่ PVC สำหรับ Deployment เราต้องสร้าง PVC เองแล้วให้ทุก replica ชี้มาที่ตัวเดียวกัน ความต่างจริงๆ จึงเป็น

	Deployment	StatefulSet
PVC	สร้างเอง ทุก replica แชร์ตัวเดียว	volumeClaimTemplates สร้างให้ แยกตัวละ replica
ชื่อ Pod	สุ่ม	คงที่ (web-0, web-1, ...)

!! ต่างกันมากคือ deployment ถ้าไม่มี PVC ตัว Pod จะขึ้นไม่ได้ และจะไม่มีการสร้าง PVC ให้, แต่ถ้าเป็น statful มันจะสร้าง PVC ให้



ข้อ 2 ถูกต้อง

Pod ชื่อ web-1 ตัวใหม่จะผูกกับ data-web-1 เสมอ นี่คือ sticky identity ของ StatefulSet คุณยังไม่ได้บอกว่าเห็นอะไรใน who.txt ถ้าข้อมูลรอด ควรเห็นบรรทัด web-1 สองบรรทัด


ข้อ 3 ถูกต้อง มีรายละเอียดเพิ่ม

เหตุผลที่คุณเดาตรงกับเจตนาออกแบบครับ k8s ไม่ลบข้อมูลของ workload แบบ stateful เอง เพื่อกันข้อมูลหายจากความพลาด มีสองเรื่องที่ควรรู้

เรื่องนี้ปรับได้ด้วย persistentVolumeClaimRetentionPolicy ใน StatefulSet (whenDeleted / whenScaled เป็น Retain หรือ Delete) ค่าเริ่มต้นคือ Retain
มีสองชั้น ชั้นแรกคือ PVC รอดหลังลบ StatefulSet ชั้นที่สองคือ reclaim policy ของ PV ถ้า StorageClass เป็น Delete ตอนคุณลบ PVC เองข้อมูลก็หายตาม (เหมือน Lab 2) ดังนั้นการปกป้องข้อมูลจริงต้องดูทั้งสองชั้น