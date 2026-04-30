# 4. File System Implementation

## ภาพรวม

บทนี้อธิบายการ implement file system จริง ตั้งแต่โครงสร้างบน disk และใน memory ไปจนถึงการ mount, การจัดการ directory, การจัดสรร blocks, และการรองรับหลาย file systems

## 1) File-System Structure

- file system อยู่บน `secondary storage` เช่น disk
- หน้าที่หลักคือจัดเก็บและจัดการไฟล์บน disk
- ข้อมูลสำคัญของไฟล์ถูกเก็บใน `File Control Block (FCB)`
- device driver เป็นชั้นที่คุยกับอุปกรณ์จริง
- file system มักแบ่งเป็นหลายชั้นเพื่อลดความซับซ้อน

## 2) File System Layers

- `Logical file system`
  - จัดการ metadata, directory, protection
  - แปลง `file name` ไปเป็น `inode` หรือข้อมูลอ้างอิงของไฟล์
- `File organization module`
  - แปลง `logical block number` เป็น `physical block number`
  - จัดการ `free space` และการจัดสรร disk
- `Basic file system`
  - จัดการ `buffers` และ `caches`
  - รับคำสั่งระดับ block
- `I/O control layer`
  - device driver คุยกับฮาร์ดแวร์จริง

ลำดับการทำงานแบบสั้น:

- ชื่อไฟล์ -> `Logical file system`
- blocks ของไฟล์ -> `File organization module`
- คำสั่งระดับ block -> `Basic file system`
- คุยกับอุปกรณ์จริง -> `I/O control layer`

ข้อดีคือแยกหน้าที่ชัดและลดความซับซ้อน แต่แลกกับ overhead เพิ่ม

## 3) File-System Implementation

- ระบบต้องมีทั้งโครงสร้างบน disk และใน memory เพื่อรองรับ system calls 
- `Boot control block`
  - เก็บข้อมูลที่ต้องใช้ตอน boot
- `Volume control block`
  - เช่น `superblock`
  - เก็บข้อมูลรวมของ volume เช่น จำนวน blocks, block size, และ free-space info
- `Directory structure`
  - map จากชื่อไฟล์ไปยัง `inode` หรือ `FCB`
- `FCB`
  - เก็บ metadata ของไฟล์ เช่น size, permission, time และตำแหน่งข้อมูล
  - ใน Unix แนวคิดนี้มักอยู่ในรูป `inode`

จำง่าย:

- `Directory` บอกว่าไฟล์ชื่อนี้คือไฟล์ไหน
- `FCB/inode` บอกว่าไฟล์นี้มีข้อมูลอะไรและเก็บอยู่ที่ไหน

## 4) In-Memory File System Structures

- เป็นโครงสร้างข้อมูลที่ OS เก็บใน `RAM` ระหว่างทำงาน
- ไม่ได้เก็บเนื้อไฟล์ทั้งหมด แต่เก็บข้อมูลที่ช่วยให้จัดการ file system ได้เร็วขึ้น
- `Mount table`
  - เก็บว่า file system ไหนถูก mount ไว้ที่ไหน
  - ช่วยให้ OS รู้ว่า path หนึ่งควรส่งไปยัง volume และชนิด file system ใด
- มักมีข้อมูลของไฟล์ที่เปิดอยู่, file handles, และ buffers/caches

flow ตอนเปิดไฟล์แบบย่อ:

1. OS ดู `mount table` และไปยัง `volume` นั้น
2. หา `directory entry` ของ file system นั้น
3. ตามไป `FCB/inode`
4. map ไปยัง disk blocks
5. อ่านผ่าน driver แล้วส่งข้อมูลกลับให้ process

flow ตอน "อ่านไฟล์" แบบเห็นภาพ:

1. process เรียก `open("file")`
2. OS ใช้ `mount table` ดูว่า path นี้อยู่บน `volume` และ file system ไหน
3. OS ค้น `directory entry` ของไฟล์
4. OS ตามไปยัง `FCB/inode` เพื่อดู metadata และตำแหน่ง blocks
5. OS ส่งกลับ `file handle` ให้ process
6. ต่อมา process เรียก `read()`
7. OS ใช้ข้อมูลใน `file handle` และ `FCB/inode` เพื่อรู้ว่าต้องอ่าน block ไหน
8. `File organization module` แปลง logical block เป็น physical block
9. `Basic file system` ขอ block ที่ต้องใช้ และเช็ก `buffer/cache` ก่อน
10. ถ้ายังไม่มีใน memory, `I/O control layer` และ device driver จะไปอ่านจาก disk
11. ข้อมูลถูกนำมาไว้ใน buffer/cache ของ kernel
12. `read()` คัดลอกข้อมูลจาก kernel buffer ไปยัง memory ของ process
13. file offset ถูกเลื่อนไปตำแหน่งถัดไป

หมายเหตุเรื่อง `MMU`:
- `MMU` ไม่ได้เป็นชั้นของ file system
- มันทำงานตอน CPU เข้าถึง `virtual address` ของ process หรือของ kernel
- ดังนั้นใน flow นี้ `MMU` จะเกี่ยวตอน process เรียก `read()`, ตอน kernel ใช้ memory/buffer, และตอนคัดลอกข้อมูลจาก kernel buffer ไปยัง memory ของ process
- ส่วนงานหาไฟล์, หา `directory entry`, และ map file blocks เป็นหน้าที่ของ file system ไม่ใช่ `MMU`

## 5) Partitions and Mounting

- `Partition` คือการแบ่ง disk ออกเป็นส่วนย่อย
- แต่ละ partition อาจมี file system ของตัวเอง หรือเป็น `raw` partition ก็ได้
- `boot block` ใช้กับการเริ่มบูตระบบ
- `root partition` คือ partition หลักที่เก็บ OS
- การ mount อาจทำอัตโนมัติหรือต้องสั่งเอง
- มักเกิดตอนบูตเครื่อง, ตาม config, ตอนผู้ใช้สั่ง mount, หรือเมื่อเสียบอุปกรณ์
- ตอน mount ระบบจะตรวจ `file system consistency`
- ถ้าถูกต้องจึงเพิ่มเข้า `mount table`

จำง่าย:

- `disk` = อุปกรณ์จริง
- `partition` = ส่วนที่แบ่งออกมา
- `file system` = วิธีจัดระเบียบไฟล์บน partition นั้น

## 6) Virtual File Systems (VFS)

- `VFS` เป็นชั้นกลางที่ทำให้หลาย file systems ใช้ system call เดียวกันได้
- app จึงเรียก `open()`, `read()`, `write()` แบบเดิมโดยไม่ต้องรู้ว่า backend เป็น `ext4`, `NTFS`, USB หรือ `NFS`
- หน้าที่ของ `VFS` คือดูว่า path นี้อยู่บน file system ไหน แล้ว dispatch ต่อไปยัง implementation ที่ถูกต้อง

## 7) Virtual File System Implementation

- object หลักที่สไลด์ยกคือ `inode`, `file`, `superblock`, และ `dentry`
- `VFS` กำหนดชุด operations ที่ object เหล่านี้ต้องรองรับ
- แต่ละ object จะมี pointer ไปยัง `function table`
- มองแบบ OOP ได้ว่า VFS กำหนด interface กลาง แล้วแต่ละ file system ไป implement เอง
- ภายนอกจึงใช้ API เหมือนกัน แต่ข้างในทำงานต่างกันได้

## 8) Directory Implementation

- วิธีพื้นฐานคือ `linear list`
  - ง่าย แต่ค้นหาช้าเพราะต้อง `linear search`
- อีกวิธีคือ `hash table`
  - ค้นหาเร็วขึ้น
  - ต้องจัดการ `collision`

สรุปสั้น:

- `linear list` = ง่ายแต่ช้า
- `hash table` = เร็วขึ้นแต่ต้องจัดการ collision
- `directory` = ชื่อไฟล์ -> `FCB/inode`

## 9) Allocation Methods

`allocation method` คือวิธีจัดสรร disk blocks ให้ไฟล์ โดย `random access` หมายถึงการกระโดดไปยังตำแหน่งที่ต้องการได้ทันที

### 9.1 Contiguous Allocation

- ไฟล์หนึ่งไฟล์กิน blocks ที่อยู่ติดกัน
- ข้อดี:
  - เร็วและโครงสร้างง่าย
  - รู้แค่ `starting block` กับ `length`
- ข้อเสีย:
  - ต้องหาพื้นที่ว่างที่ติดกันพอ
  - ต้องรู้หรือประมาณขนาดไฟล์
  - เกิด `external fragmentation`
  - อาจต้อง `compaction`

การ map address:

- ถ้า `LA` คือ logical address
- `Q = LA / block_size`
- `R = LA mod block_size`
- physical block ที่ใช้คือ `starting block + Q`
- ส่วน offset ใน block คือ `R`

### 9.2 Extent-Based Systems

- เป็น contiguous allocation แบบปรับปรุง
- `extent` คือกลุ่ม blocks ที่ต่อเนื่องกันก้อนหนึ่ง
- ไฟล์หนึ่งไฟล์อาจมีหลาย extents
- ได้ความยืดหยุ่นมากกว่า contiguous allocation ตรง ๆ
- พบใน file systems รุ่นใหม่หลายตัว

### 9.3 Linked Allocation

- แต่ละ block ของไฟล์จะมี pointer ไปยัง block ถัดไป
- ข้อดี:
  - ไม่ต้องการพื้นที่ติดกัน
  - ไม่มี `external fragmentation`
- ข้อเสีย:
  - random access ไม่ดี
  - มี overhead จาก pointer
  - ถ้า pointer เสีย อาจกระทบทั้งสายโซ่ของไฟล์

### 9.4 File-Allocation Table (FAT)

- เป็นแนวคิด linked allocation ที่ย้ายข้อมูลการ link ไปเก็บใน table กลาง
- แต่ละ entry ใน `FAT` บอกว่า block ถัดไปของ block นี้คืออะไร
- ข้อดี:
  - ตาม chain ง่ายขึ้น
  - จัดการไฟล์ได้สะดวกกว่าการเก็บ pointer ไว้ใน data block ตรง ๆ
- ข้อเสีย:
  - table อาจมีขนาดใหญ่
  - ถ้า table เสียหายจะกระทบหนัก

### 9.5 Indexed Allocation

- แต่ละไฟล์มี `index block` ของตัวเอง
- index block เก็บ pointer ไปยัง data blocks ของไฟล์
- ข้อดี:
  - รองรับ `random access`
  - ขยายไฟล์ได้โดยไม่ต้องใช้พื้นที่ติดกัน
  - ไม่มี `external fragmentation`
- ข้อเสีย:
  - มี overhead จาก index block

จำง่าย:
- `contiguous` = เก็บติดกัน
- `linked` = เก็บเป็นลูกโซ่
- `indexed` = มีสารบัญ block ของไฟล์

## 10) Indexed Allocation ขั้นต่อ

### 10.1 Single-Level Index

- ไฟล์มี index block แค่ก้อนเดียว
- ถ้าไฟล์ไม่ใหญ่มาก index block เดียวอาจพอ
- การ map คือใช้ส่วนหนึ่งของ logical address ไปหา entry ใน index table
- แล้วใช้ส่วนที่เหลือเป็น offset ใน block ข้อมูล

### 10.2 Linked Scheme ของ Index Blocks

- ถ้าไฟล์ยาวมากจน index block เดียวไม่พอ
- สามารถ link index blocks หลายก้อนต่อกันได้
- ข้อดีคือรองรับไฟล์ใหญ่แบบไม่มีขีดจำกัดตายตัวง่าย ๆ

### 10.3 Two-Level Index

- ใช้ `outer index` ชี้ไปยัง `index blocks` ชั้นใน
- เหมาะกับไฟล์ที่ใหญ่ขึ้นอีก
- ช่วยขยายขนาดไฟล์สูงสุดที่รองรับได้

### 10.4 Combined Scheme: UNIX UFS

- UFS ใช้วิธีผสมหลายแบบ
- โดยทั่วไปจะมี pointer ตรงจำนวนหนึ่งก่อน
- ถ้าไฟล์ใหญ่ขึ้นค่อยใช้ `single indirect`, `double indirect`, และ `triple indirect`
- แนวคิดคือ
  - ไฟล์เล็กเข้าถึงได้เร็ว
  - ไฟล์ใหญ่ก็ยังขยายต่อได้มาก

จำง่าย:

- `single indirect` = ผ่าน 1 ชั้น
- `double indirect` = ผ่าน 2 ชั้น
- `triple indirect` = ผ่าน 3 ชั้น
- ข้อดีคือไฟล์เล็กยังเร็ว ส่วนไฟล์ใหญ่ก็ยังขยายได้

## 11) NFS และมุมมองหลาย File Systems

- `NFS (Network File System)` คือ file system ผ่านเครือข่าย
- ทำให้ remote files ถูกใช้งานได้คล้าย local files
- `VFS` ช่วยให้ API เดิมยังใช้ได้ แม้ backend จะเป็น network file system

## 12) หมายเหตุเรื่องหัวข้อในสารบัญบท

- หน้าแรกของบทมีหัวข้ออย่าง `Free-Space Management`, `Efficiency and Performance`, `Recovery`, และ `Example: WAFL File System`
- แต่ใน PDF ชุดนี้ที่ให้มา เนื้อหาสไลด์ที่ดึงได้จบที่ `NFS Architecture`
- ดังนั้นสรุปนี้ครอบคลุมตามเนื้อหาที่มีอยู่จริงในไฟล์ต้นฉบับชุดนี้

## 13) ใจความสำคัญของบทนี้

- file system ต้องมีทั้ง layer บน disk และใน memory
- `FCB`, `superblock`, และ `mount table` เป็นโครงสร้างหลัก
- `VFS` ช่วยให้หลาย file systems ใช้ API เดียวกันได้
- การ implement directory และการจัดสรร blocks มีหลายวิธี และแต่ละวิธีมี trade-off
- `contiguous`, `linked`, `FAT`, และ `indexed` เป็นแนวทางหลักที่ต้องแยกให้ออก
- การแบ่งเป็นหลาย layer ช่วยจัดการความซับซ้อน แต่มี overhead
- การ mount ไม่ใช่แค่การต่อใช้งาน ต้องตรวจความถูกต้องของ volume ด้วย
- network file system อย่าง `NFS` ทำให้ remote files ถูกใช้งานผ่าน interface ที่คุ้นเคยได้

## Q&A

- Q: `file system layers` เรียงลำดับอย่างไร?
A: จากบนลงล่างคือ `Logical file system` -> `File organization module` -> `Basic file system` -> `I/O control layer`
- Q: แต่ละ layer ทำอะไรแบบสั้น ๆ?
A: ชั้นบนจัดการชื่อไฟล์และ metadata, ชั้นถัดมาจัด block และ free space, ชั้นต่อมาจัด buffer/cache และคำสั่งระดับ block, ชั้นล่างสุดคุยกับอุปกรณ์จริง
- Q: `FCB` คืออะไร?
A: `File Control Block` คือโครงสร้างข้อมูลที่เก็บ metadata ของไฟล์ เช่น ขนาด สิทธิ์ เวลา และตำแหน่งอ้างอิงไปยังข้อมูลจริงของไฟล์
- Q: `FCB` ต่างจาก `Directory` ยังไง?
A: `Directory` เก็บชื่อไฟล์และชี้ไปยังไฟล์นั้น ส่วน `FCB` เก็บรายละเอียดของไฟล์นั้น เช่น size, permission, time และตำแหน่ง data blocks
- Q: ทำไม `mount table` ต้องเก็บว่า mount อะไรไว้ที่ไหน?
A: เพื่อให้ OS รู้ว่า path ที่กำลังถูกใช้งานต้องส่งต่อไปยัง volume ไหน และต้องจัดการด้วย file system ชนิดใด
- Q: เหตุการณ์ไหนที่ทำให้เกิดการ `mount`?
A: มักเกิดตอนบูตเครื่อง, ตอนระบบ mount partition อัตโนมัติ, ตอนผู้ใช้สั่ง mount เอง, หรือเมื่อเสียบสื่อเก็บข้อมูลแล้วระบบ auto-mount
- Q: `In-Memory File System Structures` คืออะไร?
A: คือโครงสร้างข้อมูลของ file system ที่ OS เก็บไว้ใน RAM ระหว่างทำงาน เพื่อให้เปิดไฟล์ อ่านไฟล์ และจัดการ mount ได้เร็วขึ้น
- Q: ขอตัวอย่าง flow จริงของ `mount table` ตอนเปิดไฟล์
A: เช่นเปิด `/media/usb/report.txt` ระบบจะดู `mount table` ก่อนว่า path นี้อยู่บน USB ไหนและเป็น file system ชนิดอะไร แล้วค่อยหา directory entry, ตามไป `FCB`, หา blocks และสั่ง device driver อ่านข้อมูลจริง
- Q: ถ้าเราอ่านไฟล์ 1 ไฟล์ จะเกิดอะไรขึ้นบ้าง?
A: โดยสั้น ๆ คือ `open()` หา file system, directory entry, และ `FCB/inode` ก่อน จากนั้น `read()` จะใช้ข้อมูลนี้ไปหา blocks ของไฟล์, อ่านผ่าน buffer/cache หรือ disk, แล้วคัดลอกข้อมูลกลับไปยัง memory ของ process
- Q: `MMU` อยู่ตรงไหนใน flow การอ่านไฟล์?
A: `MMU` ไม่ได้อยู่ในชั้น file system โดยตรง แต่มาเกี่ยวตอน CPU เข้าถึง memory เช่นตอน process เรียก `read()`, ตอน kernel ใช้ buffer, และตอนคัดลอกข้อมูลกลับไปยัง memory ของ process
- Q: คำว่า `file system` ที่ OS เก็บไว้หมายถึงอะไร?
A: หมายถึงข้อมูลสำหรับจัดการระบบไฟล์ของ volume หนึ่ง ๆ เช่นของ SSD, USB หรือ CD รวมถึงชนิดของมันอย่าง `ext4` หรือ `NTFS` ไม่ได้หมายถึงเก็บทุกไฟล์ทั้งหมดไว้ใน RAM
- Q: `partition` คืออะไร?
A: คือส่วนที่ถูกแบ่งออกมาจาก disk ลูกหนึ่ง เพื่อใช้เป็นพื้นที่เก็บข้อมูลแยกกัน แต่ละ partition อาจมี file system ของตัวเองหรือเป็น raw ก็ได้
- Q: `boot block` ในหัวข้อ partitions คือ `boot control block` ไหม?
A: ใช่ ในบริบทนี้หมายถึงแนวคิดเดียวกัน โดย `boot block` เป็นคำเรียกสั้น ๆ ของ `boot control block`
- Q: `file system` อยู่ตรงไหนกันแน่?
A: มันอยู่บน partition หรือ volume ไม่ใช่ตัว disk เอง โดย disk 1 ลูกอาจถูกแบ่งเป็นหลาย partition และแต่ละ partition อาจมี file system ของตัวเอง
- Q: `VFS` คืออะไร?
A: คือชั้นกลางที่ทำให้ system calls เดียวกัน เช่น `open()` หรือ `read()` ใช้ได้กับ file systems หลายชนิด
- Q: อธิบาย `VFS` แบบง่าย ๆ ได้ไหม?
A: มันคือตัวกลางที่รับคำสั่งจาก app เช่น `open()` แล้วส่งต่อไปยัง file system ที่ถูกต้อง โดย app ไม่ต้องรู้ว่าไฟล์อยู่บนระบบไฟล์ชนิดไหน
- Q: เวลาปกติโปรแกรมคุยกับ file system ผ่าน `VFS` ใช่ไหม?
A: ใช่ ในมุมของโปรแกรมทั่วไปจะเรียกผ่าน API เดียว เช่น `open/read/write` แล้ว VFS เป็นตัวกลางคอยส่งต่อไปยัง file system จริงที่เกี่ยวข้อง
- Q: หัวข้อ `Virtual File Systems` กับ `Virtual File System Implementation` ต่างกันยังไง?
A: หัวข้อแรกเน้นแนวคิดและหน้าที่ของ VFS ส่วนหัวข้อหลังเน้นว่า VFS ภายในมี object อะไรและใช้ function tables ส่งงานอย่างไร
- Q: มอง `VFS implementation` แบบ OOP ได้ไหม?
A: ได้ค่อนข้างมาก เพราะ VFS เหมือนกำหนด interface กลางของ operations แล้วให้แต่ละ file system ไป implement รายละเอียดของตัวเอง
- Q: `linear list` กับ `hash table` ใน directory ต่างกันยังไง?
A: `linear list` ง่ายแต่ค้นหาช้า ส่วน `hash table` ค้นหาเร็วขึ้นแต่ต้องจัดการ collision
- Q: ทำไมบางสไลด์บอกว่า directory ชี้ไป data blocks?
A: เป็นการย่อภาพรวม แต่ถ้ามองให้แม่นกว่า directory มักชี้ไป `FCB` หรือ `inode` ก่อน แล้วตัวนั้นค่อยบอกต่อว่าข้อมูลจริงของไฟล์อยู่ที่ data blocks ไหน
- Q: `contiguous allocation` เด่นเรื่องอะไร?
A: เด่นเรื่องความเร็วและความง่าย เพราะเก็บไฟล์ติดกันและใช้แค่ starting block กับ length
- Q: ทำไมใน `contiguous allocation` ถึงใช้ `Q = LA / block_size` และ `R = LA mod block_size`?
A: เพราะต้องแยกให้ออกว่า logical address อยู่ใน block ลำดับที่เท่าไรของไฟล์ (`Q`) และอยู่ตรงไหนภายใน block นั้น (`R`) แล้วค่อยเอา `Q` ไปบวกกับ `starting block` เพราะ blocks ของไฟล์วางติดกัน
- Q: `extent-based systems` ดีกว่า contiguous allocation เดิมยังไง?
A: มันยังเก็บข้อมูลเป็นช่วงที่ต่อเนื่องอยู่เพื่อให้เร็ว แต่ไม่บังคับว่าทั้งไฟล์ต้องเป็นก้อนเดียว จึงขยายไฟล์ได้ยืดหยุ่นกว่าและลดปัญหาหาพื้นที่ติดกันก้อนใหญ่
- Q: `random access` คืออะไร?
A: คือการกระโดดไปอ่านหรือเขียนตำแหน่งที่ต้องการได้ทันที โดยไม่ต้องไล่จากต้นไฟล์ทีละส่วน
- Q: `linked allocation` เสียเปรียบเรื่องอะไร?
A: random access ไม่ดี และมี overhead จาก pointer ที่เชื่อม blocks
- Q: `indexed allocation` มองภาพง่าย ๆ คืออะไร?
A: คือให้แต่ละไฟล์มี `index block` ทำหน้าที่เหมือนสารบัญว่าข้อมูลของไฟล์แต่ละ block ไปอยู่ที่ disk block ไหน
- Q: ประโยคที่ว่า "ใช้ส่วนหนึ่งของ logical address ไปหา entry ใน index table แล้วใช้ส่วนที่เหลือเป็น offset" หมายถึงอะไร?
A: หมายถึงแยก logical address ออกเป็น 2 ส่วน คือเลข block ของไฟล์สำหรับ lookup ใน index table และตำแหน่งย่อยภายใน block นั้นสำหรับอ่านข้อมูลจริง
- Q: ทำไม `indexed allocation` ถึงนิยม?
A: เพราะรองรับ random access ได้ดี และไม่ต้องบังคับให้ blocks ของไฟล์อยู่ติดกัน
- Q: `two-level index` แบบสั้น ๆ คืออะไร?
A: คือมี index 2 ชั้น โดย `outer index` จะชี้ไป `index block` ย่อย แล้ว index block ย่อยค่อยชี้ไป data blocks จริง
- Q: `FAT` คืออะไร?
A: คือรูปแบบ linked allocation ที่เก็บข้อมูลการเชื่อม block ไว้ใน table กลางแทนการเก็บ pointer ไว้ใน data block โดยตรง
- Q: `UFS` ใช้วิธีแบบไหน?
A: ใช้แบบผสม โดยมี direct pointers และ indirect pointers หลายระดับเพื่อให้รองรับทั้งไฟล์เล็กและไฟล์ใหญ่
- Q: `Combined Scheme: UNIX UFS` แบบง่าย ๆ คืออะไร?
A: คือการผสม direct pointers กับ indirect pointers หลายระดับ เพื่อให้ไฟล์เล็กเข้าถึงเร็ว แต่ไฟล์ใหญ่ก็ยังขยายต่อได้
- Q: `single`, `double`, `triple indirect block` คืออะไร?
A: คือ block ที่ไม่ได้เก็บข้อมูลไฟล์โดยตรง แต่เก็บ pointer ต่อเป็นชั้น ๆ ไปยัง data blocks โดย `single` ผ่าน 1 ชั้น, `double` ผ่าน 2 ชั้น, และ `triple` ผ่าน 3 ชั้น
- Q: `NFS` คืออะไร?
A: คือ network file system ที่ทำให้เข้าถึงไฟล์บนเครื่องอื่นผ่านเครือข่ายได้คล้ายใช้ไฟล์ในเครื่องตัวเอง
