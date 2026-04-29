# 6. Kernel Module

## ภาพรวม
Kernel module คือโค้ดที่สามารถโหลดเข้าและถอดออกจาก kernel ได้ตามต้องการ เพื่อขยายความสามารถของระบบโดยไม่ต้อง reboot

## 1) What is Kernel Module?
- เป็นชิ้นส่วนโค้ดที่โหลดและ unload จาก kernel ได้
- ใช้ขยายฟังก์ชันของ kernel แบบ on demand
- มองง่าย ๆ ว่าเป็น "ปลั๊กอินของ kernel"
- คือไม่ต้องฝังทุกอย่างไว้ใน kernel ตั้งแต่แรก
- ถ้าต้องใช้ความสามารถบางอย่าง เช่น driver ของอุปกรณ์ ก็โหลด module นั้นเข้ามาได้ตอนต้องการ
- ตัวอย่างการใช้งาน:
  - hardware drivers
  - special APIs
  - ส่วนขยายอื่น ๆ ของ kernel

flow แบบง่าย:
1. ตอนแรก module ยังไม่อยู่ใน kernel
2. ผู้ใช้หรือระบบสั่งโหลด module เช่นด้วย `insmod` หรือ `modprobe`
3. kernel นำโค้ดของ module เข้าไปในหน่วยความจำของ kernel
4. `init_module()` หรือฟังก์ชันเริ่มต้นของ module ถูกเรียก
5. module ลงทะเบียนความสามารถของตัวเอง เช่น driver หรือ file operations
6. หลังจากนั้น kernel เรียกใช้ module นี้ได้เมื่อมี event ที่เกี่ยวข้อง
7. ถ้าไม่ต้องใช้แล้ว สามารถสั่งถอดด้วย `rmmod`
8. kernel จะเรียก `cleanup_module()` ก่อนเอา module ออก

ตัวอย่างภาพจริง:
- เสียบอุปกรณ์บางอย่าง
- kernel ต้องใช้ driver ของอุปกรณ์นั้น
- จึงโหลด kernel module ที่เกี่ยวข้อง
- module ลงทะเบียนตัวเอง
- จากนั้น kernel ใช้ module นี้คุยกับอุปกรณ์ได้

ตัวอย่าง hardware driver:
- `printer driver`
  - ทำให้ kernel ส่งข้อมูลไปยังเครื่องพิมพ์ได้
- `disk driver`
  - ทำให้ kernel อ่าน/เขียนข้อมูลกับ HDD หรือ SSD ได้
- `network card driver`
  - ทำให้ระบบส่งและรับข้อมูลผ่าน LAN/Wi-Fi ได้
- `USB driver`
  - ช่วยให้ระบบคุยกับอุปกรณ์ USB ได้
- `keyboard / mouse driver`
  - รับ input จากอุปกรณ์แล้วส่งต่อให้ระบบ

## 2) ข้อควรระวัง
- ไม่มี standard C library ใน kernel
  - ไม่มี `printf`, `scanf`, `gets`
- เครื่องมือ debug มีจำกัด
- ต้องใช้ kernel functions เท่านั้น
- ฟังก์ชันที่ใช้บ่อยคือ `printk`

รูปแบบการใช้งาน `printk`:
- `printk( [LEVEL] const char *format_string, ... );`

ระดับข้อความที่พบบ่อย:
- `KERN_EMERG`
- `KERN_ALERT`
- `KERN_CRIT`
- `KERN_ERR`
- `KERN_WARNING`
- `KERN_NOTICE`
- `KERN_INFO`
- `KERN_DEBUG`
- `KERN_DEFAULT`
- `KERN_CONT`

## 3) ตัวอย่าง Kernel Module แรก
- module ตัวอย่างมักประกอบด้วย
  - `MODULE_LICENSE`
  - `MODULE_AUTHOR`
  - `MODULE_DESCRIPTION`
  - `init_module()`
  - `cleanup_module()`
- `init_module()` ใช้ตอนโหลด module
- `cleanup_module()` ใช้ตอนถอด module
- มักใช้ `printk(KERN_INFO ...)` เพื่อพิมพ์ข้อความลง kernel log

## 4) การ Build และ Commands ที่ใช้บ่อย
- `Makefile` สำหรับ module มักเรียก build system ของ kernel โดยตรง
- ต้องมี kernel headers ติดตั้งไว้ เช่น `linux-kernel-headers` หรือ `linux-libc-dev`

คำสั่งที่ใช้บ่อย:
- `lsmod`
  - ดูสถานะ modules ที่โหลดอยู่
- `insmod`
  - โหลด module เข้า kernel
- `rmmod`
  - ถอด module ออกจาก kernel
- `modprobe`
  - เพิ่มหรือลบ module พร้อมจัดการ dependency
- `modinfo`
  - ดูข้อมูล module
- `dmesg`
  - ดู kernel ring buffer

## 5) Device Drivers และ Device File
- device file คือ interface ระหว่าง user space กับ device driver ผ่าน file system
- ใช้ file permissions ควบคุมสิทธิ์ของ device
- ใช้ file operations ในการ access device

ตัวอย่าง:
- เขียนไปที่ `/dev/lp0` อาจส่งข้อมูลไปยัง printer
- terminal device ก็เข้าถึงได้ผ่าน device file เช่น `/dev/pts/0`

## 6) Character vs Block Devices
- `Character device`
  - ส่งและรับข้อมูลทีละตัวอักษร
  - seek ไม่ได้
  - ตัวอย่างเช่น serial หรือ parallel port
- `Block device`
  - ส่งข้อมูลเป็น block
  - seek ได้
  - ตัวอย่างเช่น disk และ USB camera

## 7) Major/Minor Device Number
- device แบ่งเป็นกลุ่มด้วย `major number`
- major number เดียวกันหมายถึงใช้ driver เดียวกัน
- `minor number` ใช้แยก device ย่อยภายในกลุ่มเดียวกัน

ตัวอย่าง:
- `/dev/sda`, `/dev/sda1`, `/dev/sda2` อยู่ในกลุ่ม major เดียวกันของ disk
- `/dev/pts/0` เป็น character device ของ pseudo terminal

มองให้ง่าย:
- `major number` = บอกว่า "ต้องเรียก driver ตัวไหน"
- `minor number` = บอกว่า "เป็นอุปกรณ์ย่อยตัวไหนของ driver นั้น"

ตัวอย่างเห็นภาพ:
- สมมติ driver ของ disk ใช้ `major = 8`
- `/dev/sda` อาจเป็น `8,0`
- `/dev/sda1` อาจเป็น `8,1`
- `/dev/sda2` อาจเป็น `8,2`
- ความหมายคือทั้งหมดใช้ disk driver ตัวเดียวกัน
- แต่ `minor` ต่างกันเพื่อบอกว่าเป็นทั้งดิสก์ทั้งลูก หรือเป็น partition ไหน

อีกตัวอย่าง:
- `/dev/pts/0` อาจเป็น `136,0`
- `/dev/pts/1` อาจเป็น `136,1`
- `major 136` บอกว่าใช้ driver ของ pseudo terminal
- `minor 0` กับ `1` บอกว่าเป็น terminal คนละตัว

สรุปภาพจำ:
- `major` = แผนก
- `minor` = หมายเลขคิวในแผนกนั้น

## 8) Creating a Device
- ใช้ `mknod` สร้าง device file ได้
- ตัวอย่างเช่น `mknod /dev/osinfo c 250 0`
- `c` หมายถึง character device

## 9) File Operations in Drivers
- driver มักกำหนดชุด `file_operations`
- ตัวอย่างฟังก์ชันในโครงสร้างนี้ เช่น
  - `read`
  - `write`
  - `open`
  - `release`
  - `ioctl`
  - `mmap`
- ถ้าไม่ใช้ operation ใด ให้ตั้งเป็น `NULL`
- มักเขียนด้วย C99 syntax เพื่อกำหนดฟังก์ชันที่ต้องใช้เท่านั้น

## 10) ใจความสำคัญของบทนี้
- kernel module ช่วยเพิ่มความสามารถของ kernel โดยไม่ต้อง reboot
- การเขียนใน kernel ต้องระวังมาก เพราะเครื่องมือและ library จำกัด
- device driver มักสื่อสารผ่าน device file และ `file_operations`
- `printk`, `lsmod`, `insmod`, `rmmod`, และ `dmesg` เป็นเครื่องมือสำคัญพื้นฐาน

## Q&A
- Q: `kernel module` คืออะไรแบบง่าย ๆ?
  A: มันเหมือนปลั๊กอินของ kernel คือโค้ดที่โหลดเข้า kernel ตอนต้องใช้ เพื่อเพิ่มความสามารถโดยไม่ต้อง reboot เครื่อง
- Q: flow ของ `kernel module` แบบง่าย ๆ เป็นยังไง?
  A: โหลด module เข้า kernel -> เรียกฟังก์ชันเริ่มต้น -> ลงทะเบียนความสามารถ -> kernel ใช้งานมัน -> ตอนถอดออกเรียก cleanup แล้วค่อย unload
- Q: `hardware driver` มีตัวอย่างอะไรบ้าง?
  A: เช่น driver ของ printer, disk, network card, USB, keyboard และ mouse
- Q: `major/minor device number` มองภาพง่าย ๆ คืออะไร?
  A: `major` บอกว่าจะใช้ driver ตัวไหน ส่วน `minor` บอกว่าเป็นอุปกรณ์ย่อยตัวไหนภายใต้ driver นั้น เช่น `/dev/sda1` กับ `/dev/sda2` ใช้ driver เดียวกันแต่เป็นคนละ partition
