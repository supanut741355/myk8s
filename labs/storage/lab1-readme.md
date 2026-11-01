# Lab 1: ข้อมูลหายตอน Pod ถูกลบs

## set up
kubectl create ns storage

kubectl apply -f lab1.yaml -n storage
kubectl exec -n storage lab1 -- cat /data/note.txt        # เห็น hello



# ทดลอง 1: ฆ่า container ข้างใน (Pod ยังอยู่)
kubectl exec -n storage lab1 -- sh -c "kill 1"
kubectl get pod lab1 -n storage                            # RESTARTS เพิ่มเป็น 1
kubectl exec -n storage lab1 -- cat /data/note.txt         # ไฟล์ยังอยู่


# ทดลอง 2: ลบทั้ง Pod
kubectl exec -n storage lab1 -- sh -c "echo edited >> /data/note.txt" #แก้ข้อมูลก่อนลบ pod
kubectl delete pod lab1 -n storage 
kubectl apply -f ./lab1.yaml -n storage 
kubectl exec -n storage lab1 -- cat /data/note.txt         # สร้างใหม่จากคำสั่ง echo ใน YAML ไม่ใช่ข้อมูลเก่า

--- Quiz
1. ทำไมทดลอง 1 ไฟล์ยังอยู่ แต่ทดลอง 2 ข้อมูลที่เราเพิ่มเองหายไป?
ANS: ทดลอง 1 ที่ยังอยู่ เพราะไฟล์ถูกเก็บใน storage ของ pod เพราะเราไม่ได้ลบ pod แต่ทดลอง 2 เราลบ pod ทำให้ไฟ์หาย

emptyDir ผูกกับ ชีวิตของ Pod ไม่ใช่ชีวิตของ container ดังนั้น container ตายแล้วถูก restart ข้อมูลยังอยู่ แต่พอ Pod ถูกลบ ข้อมูลหายไปด้วย

emptyDir คืออะไร???
-> emptyDir คือ volume ชนิดหนึ่งที่ k8s สร้างเป็น โฟลเดอร์ว่างๆ บน node ที่ Pod รันอยู่ ตอน Pod เริ่มทำงาน แล้วลบทิ้งตอน Pod ถูกลบ, ลองเทียบกับ /data ใน lab เมื่อกี้ เราเขียนไฟล์ลง /data ที่ mount มาจาก emptyDir ซึ่งก็คือโฟลเดอร์ชั่วคราวบน node นั่นเอง

2. ถ้าเราต้องการเก็บฐานข้อมูล emptyDir เหมาะไหม เพราะอะไร?
ANS: ไม่เหมาะกับการทำฐานข้อมูล เพราะข้อมูลหายตอน Pod ถูกลบหรือถูกย้ายไปรันที่ node อื่น ส่วนแค่ container crash แล้ว restart ข้อมูลยังอยู่ (เหมือนทดลอง 1) แต่ฐานข้อมูลจริงต้องรอดทั้งสองกรณี เลยต้องมี storage ที่ แยกอายุออกจาก Pod