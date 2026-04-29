# 5. FUSE

## ภาพรวม
`FUSE (Filesystem in Userspace)` คือแนวทางสร้าง file system ใน `user space` แทนการเขียนอยู่ใน kernel โดยมี kernel module ทำหน้าที่เป็นสะพานเชื่อมระหว่าง system calls กับ process ของ file system ที่เราพัฒนา

## 1) ทำไมต้อง Userspace
การเขียน code ใน kernel ยากและเสี่ยงกว่า

ปัญหาหลัก:
- ไม่มี `libc` ปกติให้ใช้
- debug ยาก
- ผิดพลาดอาจทำให้ `kernel panic`
- ทดลองแก้ไขแต่ละครั้งต้นทุนสูง

ดังนั้น FUSE ช่วยให้พัฒนา file system ได้เร็วขึ้นและปลอดภัยขึ้นในเชิง workflow

## 2) FUSE คืออะไร
- เป็น framework สำหรับ Unix-like systems
- ให้ผู้ใช้ทั่วไปสร้าง file system ของตัวเองได้โดยไม่ต้องแก้ kernel code โดยตรง
- file system logic รันใน user space
- kernel module ของ FUSE เป็น bridge ไปยัง kernel interfaces

จุดเด่น:
- เลือกภาษาและเครื่องมือได้ยืดหยุ่น
- เหมาะกับการทดลอง prototype และงานวิจัย

## 3) ภาพการทำงาน
แนวคิดโดยย่อ:
1. โปรแกรมเรียก file system API เช่น open/read/write
2. kernel ส่งคำขอผ่าน FUSE driver
3. process ของ file system ใน user space รับคำขอไปจัดการ
4. ส่งผลกลับมายัง kernel และ user program

จึงเหมือนมี file system เสมือนคั่นอยู่ระหว่างโปรแกรมกับ backend จริง

## 4) งานที่ file system ต้องรับผิดชอบ
การสร้าง file system จริงไม่ใช่แค่อ่าน/เขียนไฟล์ แต่ยังมีเรื่อง เช่น
- performance
- metadata
- permissions
- security / encryption
- maximum file size
- deduplication
- compression
- snapshots

FUSE ช่วยเรื่อง workflow แต่ไม่ได้ลดความยากของการออกแบบ file system เองทั้งหมด

## 5) Operations สำคัญของ FUSE
ตัวอย่าง operations ที่มักต้อง implement:
- `getattr`
- `readlink`
- `mknod`
- `mkdir`
- `unlink`
- `rmdir`
- `symlink`
- `rename`
- `open`
- `read`
- `write`
- `truncate`
- `statfs`
- `flush`
- `opendir`
- `readdir`
- `releasedir`
- `init`

สรุปคือ FUSE ให้ API ที่ใกล้กับงาน file system จริงมาก

## 6) ตัวอย่าง flow ของคำสั่ง
สไลด์ยกตัวอย่างว่า command ง่าย ๆ หนึ่งคำสั่ง อาจเรียกหลาย operations ต่อกัน เช่น

- `cat a.txt`
  มักผ่าน `getattr -> open -> read -> release`
- `echo "a" > a.txt`
  อาจมี `getattr -> create/open -> write -> flush -> release`
- `mkdir demo`
  มักมี `getattr -> mkdir`
- `ls demo/`
  มักมี `getattr -> opendir -> readdir -> releasedir`

จุดนี้สำคัญเพราะทำให้เห็นว่าการ implement file system ต้องเข้าใจ syscall flow ด้วย

## 7) การใช้งานจริง
มี implementation จริงจำนวนมาก เช่น
- `SSHFS`
- `EncFS`
- `NTFS-3G`
- `FTPFS`
- `WikipediaFS`

จึงไม่ใช่แค่ของเล่น แต่ใช้ทำระบบจริงได้ในหลายบริบท

## 8) ใจความสำคัญของบทนี้
- FUSE ทำให้พัฒนา file system ใน user space ได้
- ช่วยลดความเสี่ยงจากการพัฒนาใน kernel
- API ของ FUSE ครอบคลุมงาน file system สำคัญจำนวนมาก
- ต้องเข้าใจ mapping ระหว่าง user commands กับ file operations

