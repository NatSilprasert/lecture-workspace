# 5. FUSE

## ภาพรวม

FUSE (`Filesystem in Userspace`) คือแนวทางสร้าง file system โดยย้าย logic ส่วนใหญ่ไปทำใน user space แทน kernel space ทำให้พัฒนาและ debug ง่ายขึ้น

## 1) ทำไมต้องใช้ Userspace

- `Userspace` คือพื้นที่ที่โปรแกรมทั่วไปของผู้ใช้รันอยู่
- แยกจาก `kernel space` ซึ่งเป็นพื้นที่ของระบบปฏิบัติการและสิทธิ์ระดับลึก
- ถ้าโปรแกรมใน userspace พัง มักกระทบแค่ process นั้น
- แต่ถ้าโค้ดใน kernel พัง อาจทำให้ทั้งระบบค้างหรือเกิด `kernel panic`
- การเขียนโค้ดใน kernel ยากกว่า เพราะ debug ยาก, ใช้ library ได้จำกัด, และพลาดแล้วกระทบทั้งระบบ

## 2) FUSE คืออะไร

- FUSE เป็น interface สำหรับสร้าง file system บน Unix-like systems
- ผู้ใช้ที่ไม่ใช่ privileged user ก็สร้าง file system ของตัวเองได้
- โค้ด file system รันใน `user space`
- มี `FUSE module` ใน kernel ทำหน้าที่เป็น bridge ไปยัง kernel interface จริง

ประโยชน์หลัก:

- พัฒนา file system ได้เร็วขึ้น
- ไม่ต้องแก้ kernel code โดยตรง
- เหมาะกับการทดลอง แนวคิดใหม่ หรือ file system เฉพาะงาน

## 3) How It Works

- app เรียก file system API ตามปกติ
- `FUSE module` ฝั่ง kernel รับคำสั่งและส่งต่อไปยัง process ของ file system ใน user space
- process นั้นทำ logic ของ file system แล้วส่งผลลัพธ์กลับมา
- มีการใช้งานได้บนหลายระบบ เช่น Linux, BSD, macOS, Android และ Minix
- มี binding หลายภาษา เช่น C, Python, Objective-C, Ruby, Java และ C#

## 4) ตัวอย่าง Operations ที่ FUSE รองรับ
- ตัวอย่างที่พบบ่อยคือ `getattr`, `open`, `read`, `write`, `mkdir`, `readdir`, `unlink`, `rename`, `symlink`, `link`, `chmod`, และ `statfs`
- แนวคิดคือ FUSE เปิดให้เรา implement พฤติกรรมของ file system ผ่าน callback ใน user space

## 5) ตัวอย่างแนวคิดการใช้งาน

- ใช้ทำ file system แบบ cloud, encrypted, FTP, IMAP, Git, SSH หรือ YouTube-backed file system ได้
- เหมาะกับงานที่ต้องการทดลอง logic ของ file system โดยไม่ต้องแตะ kernel
- ถ้าต้องการประสิทธิภาพหรือความปลอดภัยระดับ kernel อาจยังต้องพิจารณาข้อจำกัดของ user space

## 6) ใจความสำคัญของบทนี้

- FUSE ทำให้สร้าง file system ได้ใน user space
- ช่วยลดความเสี่ยงและความยากของ kernel development
- kernel module ทำหน้าที่เป็น bridge ระหว่าง API และ user-space implementation
- เหมาะกับการพัฒนา file system แบบเฉพาะทางหรือทดลองแนวคิดใหม่

## Q&A

- Q: `userspace` คืออะไร?
A: คือพื้นที่ที่โปรแกรมทั่วไปของผู้ใช้รันอยู่ แยกจาก `kernel space` และถ้าโปรแกรมพังมักไม่พาทั้งระบบล้ม
- Q: kernel module ที่เป็นตัวกลางใน FUSE คืออะไร?
A: ในบริบทนี้หมายถึง `FUSE module` ฝั่ง kernel ที่คอยเชื่อมระหว่าง kernel interface กับ file system logic ที่รันอยู่ใน userspace
