# 5. FUSE

## ภาพรวม

FUSE (`Filesystem in Userspace`) คือแนวทางสร้าง file system โดยย้าย logic ส่วนใหญ่ไปทำใน user space แทน kernel space ทำให้พัฒนาและ debug ง่ายขึ้น

## 1) ทำไมต้องใช้ Userspace

- `Userspace` คือพื้นที่ที่โปรแกรมทั่วไปของผู้ใช้รันอยู่
- แยกจาก `kernel space` ซึ่งเป็นพื้นที่ของระบบปฏิบัติการและสิทธิ์ระดับลึก
- ถ้าโปรแกรมใน userspace พัง มักกระทบแค่ process นั้น
- แต่ถ้าโค้ดใน kernel พัง อาจทำให้ทั้งระบบค้างหรือเกิด `kernel panic`
- การเขียนโค้ดที่ดีไม่ง่าย
- การเขียนโค้ดใน kernel ยากกว่าอีก เพราะ
  - ไม่มี `libc` แบบปกติ เช่น `printf`, `stdio`
  - debug ยาก
  - อาจต้อง reboot บ่อย
  - ถ้าพลาดอาจเจอ `kernel panic`

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

- คำสั่ง file system API จะถูกส่งไปยัง process ใน user space
- `FUSE module` ฝั่ง kernel ทำหน้าที่เป็นตัวกลาง
- มีการใช้งานได้บนหลายระบบ เช่น Linux, BSD, macOS, Android และ Minix
- มี binding หลายภาษา เช่น C, Python, Objective-C, Ruby, Java และ C#

## 4) ตัวอย่าง Operations ที่ FUSE รองรับ

- `getattr`
  - อ่าน attributes ของไฟล์
- `readlink`
  - อ่าน target ของ symbolic link
- `mknod`
  - สร้าง file node
- `mkdir`
  - สร้าง directory
- `unlink`
  - ลบไฟล์
- `rmdir`
  - ลบ directory
- `symlink`
  - สร้าง symbolic link
- `rename`
  - เปลี่ยนชื่อไฟล์
- `link`
  - สร้าง hard link
- `chmod`
  - เปลี่ยน permission bits
- `chown`
  - เปลี่ยน owner และ group
- `truncate`
  - เปลี่ยนขนาดไฟล์
- `open`
  - เปิดไฟล์
- `read`
  - อ่านข้อมูลจากไฟล์ที่เปิดอยู่
- `write`
  - เขียนข้อมูลลงไฟล์ที่เปิดอยู่
- `statfs`
  - ดูสถิติของ file system
- `flush`
  - flush ข้อมูลที่ cached อยู่
- `opendir`
  - เปิด directory
- `readdir`
  - อ่านรายการใน directory
- `releasedir`
  - ปลดล็อกหรือปิด directory
- `fsyncdir`
  - synchronize directory contents
- `init`
  - เริ่มต้น file system
- `lock`
  - ทำ POSIX file locking

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

