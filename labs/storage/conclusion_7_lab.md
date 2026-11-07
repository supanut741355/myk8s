สรุปที่เรียนมา
Lab	แก่นของเรื่อง
1	emptyDir อายุเท่า Pod
2	PVC/PV/StorageClass, dynamic provisioning
3-4	Reclaim policy, กู้ PV ที่ Released
5	Access modes (RWO ระดับ node, RWOP ระดับ Pod)
6-7	StatefulSet, PVC แยกตัว, จับคู่ด้วยชื่อ, retention policy

ตอนนี้คุณครอบคลุมแกนหลักของ storage ใน k8s แล้ว ขั้นต่อไปเลือกได้สองทาง

VolumeSnapshot: ทำ backup ระดับ volume และกู้คืนเป็น PVC ใหม่ ต่อยอดจากเรื่อง Retain ไม่ใช่ backup ได้โดยตรง
CSI: ดูว่า provisioner เบื้องหลังทำงานอย่างไร และทำไมบาง access mode/ฟีเจอร์ถึงใช้ได้เฉพาะบาง driver (เชื่อมกับเรื่องที่เจอใน Lab 5)
