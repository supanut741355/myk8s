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
    Warning  FailedScheduling  2m2s                 default-scheduler  0/1 nodes are available: 1 node(s) unavailable due to PersistentVolumeClaim with ReadWriteOncePod access mode already in-use by another pod. no new claims to deallocate, preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod

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



คำถามให้คิด
ในข้อ 2 ข้อความใน Events ของ lab5-b บอกว่าอะไร และทำไม Pod ตัวที่สองถึงไม่ขึ้น?
ในข้อ 3 ทำไมสอง Pod ถึงใช้ RWO ร่วมกันได้ (ใบ้: ดูคอลัมน์ NODE)?
ถ้าเรามี Deployment ที่ replicas: 3 ใช้ PVC แบบ RWO เดียวกัน บน cluster ที่มี 3 node จะเกิดปัญหาอะไรได้บ้าง? (ข้อ 3 เป็นโจทย์ที่ชี้ไปเรื่อง StatefulSet ซึ่งจะเป็น Lab 6 ครับ)

// ลองตอบ
1. Warning  FailedScheduling  72s (x2 over 72s)  default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. not found, เพราะ ตัว PVC เป็น RWOncePod ซึ่งอนุญาติให้แค่ pod เดียวที่สามารถ mount volume หามันได้, ซึ่งเมื่อ pod mount volume ไม่ได้เลยไม่สามารถ run ได้

2. ที่ใช้ร่วมกันได้ คิดว่าเป็นเพราะ ทั้ง 2 pod อยู่ใน node เดียวกัน ซึ่งเป็นไปตามหลักคือ หลายๆ pod ใน node สามารถต่อ PVC เดียวกันได้ แต่ถ้า pod อยู่คนละ node จะไม่สามารถต่อมาที่ vpc เดียวกันได้


3. คิดว่าจะเกิดปัญหาที่มี 1 pod สามารถต่อ vpc และสามารถ running ได้ แต่อีก 2 pod จะติดสถานะ pending เพราะไม่สามารถต่อ PVC ได้ เนื่องจากอยู่กันคนละ node 


// เฉลย


1. สิ่งที่ได้เรียนรู้จาก Lab 5:

ReadWriteOncePod จำกัดที่ระดับ Pod (Pod ตัวที่สองจะค้างที่ Pending)
ReadWriteOnce จำกัดที่ระดับ node (หลาย Pod บน node เดียวกันใช้ร่วมกันได้)
การอ่าน Event ต้องดูข้อความล่าสุด เพราะสถานะเปลี่ยนได้ตามเวลา

2. RWO จำกัดที่ระดับ node ไม่ใช่ Pod

3. ใกล้เคียง แต่มีสองจุดที่ต้องปรับ
อาจไม่เกิดปัญหาเลยก็ได้ ถ้า scheduler บังเอิญวางทั้ง 3 Pod บน node เดียวกัน ทุกตัวจะ Running พร้อมกัน ซึ่งอันตรายกว่า เพราะ 3 Pod จะเขียนดิสก์ก้อนเดียวกันพร้อมกัน ฐานข้อมูลอาจพังได้
สถานะที่เจอบน cloud มักไม่ใช่ Pending แต่เป็น ContainerCreating ค้าง พร้อม Event Multi-Attach error เพราะ Pod ถูกจัดลง node แล้ว แต่ดิสก์ยังต่ออยู่กับอีก node

และที่ลึกกว่านั้นคือ ต่อให้ต่อได้ replica ทั้ง 3 ตัวก็ไม่ควร แชร์ข้อมูลก้อนเดียวกัน ฐานข้อมูลแบบ cluster ต้องการให้แต่ละตัวมีดิสก์ของตัวเอง ซึ่ง Deployment ทำให้ไม่ได้ ต้องใช้ StatefulSet




