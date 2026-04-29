# 6. Kernel Module

## ภาพรวม
บทนี้แนะนำ `kernel module` ซึ่งเป็นชิ้นส่วนของ code ที่โหลดและถอดออกจาก kernel ได้ตามต้องการ เพื่อขยายความสามารถของระบบโดยไม่ต้อง reboot

## 1) Kernel Module คืออะไร
- เป็น code ที่ load/unload เข้า kernel ได้
- ใช้เพิ่มความสามารถของ kernel แบบยืดหยุ่น
- ตัวอย่างการใช้งาน:
  - hardware drivers
  - special APIs
  - ฟังก์ชันเสริมอื่น ๆ ของระบบ

## 2) ทำไมต้องใช้ Module
ข้อดีหลัก:
- ขยาย kernel ได้โดยไม่ต้อง compile ใหม่ทั้งระบบ
- ไม่ต้อง reboot ทุกครั้งที่เพิ่มความสามารถ
- เหมาะกับ device drivers ที่ต้องเปิด/ปิดตามการใช้งาน

## 3) ข้อควรระวังในการพัฒนา
การเขียน kernel code ยากกว่า user-space program มาก

ข้อจำกัดสำคัญ:
- ไม่มี standard C library ปกติ
- ใช้ `printf`, `scanf`, `gdb` แบบปกติไม่ได้
- ต้องใช้ kernel functions เท่านั้น
- error อาจกระทบทั้งระบบ

## 4) printk
`printk()` คือเครื่องมือพื้นฐานสำหรับแสดงข้อความจาก kernel

รูปแบบ:
- `printk([LEVEL] "message ...");`

ตัวอย่างระดับ log:
- `KERN_ERR`
- `KERN_WARNING`
- `KERN_INFO`
- `KERN_DEBUG`

ใช้คู่กับ `dmesg` เพื่อดูข้อความใน kernel ring buffer

## 5) โครงสร้าง Module เบื้องต้น
ตัวอย่างขั้นต่ำมักมี:
- `init_module()` สำหรับตอนโหลด
- `cleanup_module()` สำหรับตอนถอดออก

แนวคิด:
- ตอน init อาจ register driver หรือแสดง log
- ตอน cleanup ต้องคืนทรัพยากรที่จองไว้

## 6) การ build และคำสั่งพื้นฐาน
สไลด์ยกตัวอย่าง Makefile สำหรับ build module กับ kernel headers

คำสั่งที่ควรรู้:
- `lsmod` ดู modules ที่โหลดอยู่
- `insmod` โหลด module
- `rmmod` ถอด module
- `modprobe` จัดการ module พร้อม dependency
- `modinfo` ดูข้อมูล module
- `dmesg` ดู log ของ kernel

## 7) Device Drivers และ Device Files
ใน Linux อุปกรณ์มักถูกเข้าถึงผ่าน `device file`

แนวคิด:
- ใช้ file permissions คุมสิทธิ์
- ใช้ file operations เป็น interface เข้าถึง device

ตัวอย่าง:
- เขียนไปที่ `/dev/lp0` คือส่งข้อมูลไป printer

## 8) Character vs Block Devices
`Character device`
- รับส่งข้อมูลทีละตัว/stream
- `seek` ไม่สะดวกหรือทำไม่ได้
- ตัวอย่างเช่น serial/parallel port

`Block device`
- รับส่งข้อมูลเป็น blocks
- `seek` ได้
- ตัวอย่างเช่น disk

## 9) Major / Minor Number
- อุปกรณ์ใช้ `major number` เพื่อบอกว่าใช้ driver ตัวไหน
- `minor number` ใช้แยก instance ภายใต้ driver เดียวกัน

ตัวอย่าง:
- disk เดียวกันแต่หลาย partition อาจ major เท่ากัน แต่ minor ต่างกัน

## 10) file_operations
driver มัก expose งานผ่าน `struct file_operations`

ฟังก์ชันสำคัญ เช่น
- `open`
- `read`
- `write`
- `release`
- `ioctl`
- `mmap`

แนวคิดในสไลด์:
- implement เฉพาะ operations ที่จำเป็น
- ส่วนอื่นปล่อยเป็น `NULL` ได้
- นิยมใช้ designated initializer แบบ C99 ให้อ่านง่ายขึ้น

## 11) Register Character Device
การทำ char driver ต้อง register/unregister กับ kernel เช่น
- `register_chrdev(...)`
- `unregister_chrdev(...)`

จากนั้นจึงสร้าง device file เช่นผ่าน `mknod`

## 12) ใจความสำคัญของบทนี้
- kernel module ทำให้ขยาย kernel ได้แบบยืดหยุ่น
- การพัฒนาใน kernel ต้องระวังมากกว่าปกติ
- `printk`, `lsmod`, `insmod`, `rmmod`, `dmesg` เป็นเครื่องมือพื้นฐาน
- device drivers เชื่อมโลกของ kernel กับ device file ผ่าน `file_operations`

