# 4. File-System Implementation

## ภาพรวม
บทนี้อธิบายฝั่ง implementation ของ file system ตั้งแต่โครงสร้างเป็นชั้น ๆ, metadata, การ mount, `VFS`, วิธีจัดสรร blocks, การจัดการ free space, ไปจนถึง recovery และตัวอย่างระบบจริง

## 1) File-System Structure
file system ทำหน้าที่เชื่อมระหว่างมุมมองเชิงตรรกะของ file กับการเก็บข้อมูลจริงบน secondary storage

องค์ประกอบสำคัญ:
- file system interface
- logical file system
- file-organization module
- basic file system
- I/O control และ device drivers

ข้อดีของการแบ่งชั้น:
- ลดความซับซ้อน
- แยกหน้าที่ชัด

ข้อเสีย:
- มี overhead เพิ่มขึ้นบ้าง

## 2) Metadata และโครงสร้างสำคัญ
ตัวอย่างข้อมูลที่ระบบต้องเก็บ:
- `boot control block`
- `volume control block` หรือ `superblock`
- directory structure
- `File Control Block (FCB)` ของแต่ละไฟล์
- open-file tables และ buffers

สิ่งเหล่านี้ทำให้ระบบรู้ว่าไฟล์อยู่ไหน มีขนาดเท่าไร และจะเข้าถึงอย่างไร

## 3) Mounting และ Open File
- ตอน mount ระบบจะตรวจความสอดคล้องของ file system ก่อน
- ถ้าผ่านจึงเพิ่มเข้า `mount table`
- เมื่อ `open` file ระบบจะสร้าง handle และเก็บสถานะที่จำเป็นไว้ใน memory

## 4) Virtual File System (VFS)
`VFS` เป็นชั้นกลางที่ช่วยให้ system calls เดียวกันใช้กับ file systems หลายชนิดได้

ข้อดี:
- API ฝั่ง user/program เหมือนเดิม
- เปลี่ยน implementation ใต้ชั้นได้
- เหมาะกับระบบที่รองรับ local และ remote file systems หลายแบบ

## 5) Directory Implementation
directory อาจ implement ได้หลายแบบ เช่น
- linear list
- โครงสร้างที่ค้นหาเร็วขึ้น เช่น tree

โจทย์หลักคือ balance ระหว่าง:
- เวลา search
- เวลา insert/delete
- พื้นที่ที่ใช้

## 6) Allocation Methods
การจัดสรร block ของไฟล์มีหลายวิธี

### Contiguous Allocation
- ไฟล์อยู่เป็นก้อนต่อเนื่อง
- อ่านต่อเนื่องเร็ว
- random access ง่าย
- แต่ขยายไฟล์ยาก และมี external fragmentation

### Extent-Based Allocation
- แบ่งไฟล์เป็นหลาย `extents`
- ยืดหยุ่นกว่าการ contiguous แบบก้อนเดียว
- ยังได้ข้อดีเรื่อง locality ในระดับหนึ่ง

### Linked Allocation
- แต่ละ block ชี้ไป block ถัดไป
- ไม่มี external fragmentation
- เหมาะกับ sequential access
- random access ช้า

### Indexed Allocation
- มี index block เก็บ pointer ไป data blocks
- random access ดี
- ไม่มี external fragmentation แบบ contiguous
- แต่มี overhead ของ index block

## 7) Free-Space Management
ระบบต้องรู้ว่า block ไหนยังว่างอยู่

วิธีที่ใช้ได้ เช่น
- bit map / bit vector
- linked list
- grouping
- counting

การเลือกวิธีขึ้นกับ trade-off ระหว่างความง่าย ความเร็ว และพื้นที่

## 8) Efficiency, Recovery, และ Reliability
ประเด็นสำคัญ:
- buffer/cache ช่วยลด I/O
- ต้องมี consistency checking
- ต้องมี recovery หลัง crash
- ระบบจริงอาจใช้ logging หรือเทคนิคเฉพาะเพื่อกู้สภาพ file system

## 9) NFS และ WAFL
- `NFS` เป็นตัวอย่างของ remote file system architecture
- `WAFL` เป็นตัวอย่าง file system จริงที่ออกแบบเพื่อประสิทธิภาพและการจัดการข้อมูล

บทนี้ไม่ได้ต้องให้จำ implementation ทุกบรรทัด แต่ควรเข้าใจว่าระบบจริงมีหลายชั้นและมี trade-off ตลอดทาง

## 10) ใจความสำคัญของบทนี้
- file system ไม่ใช่แค่ directory + file name แต่มี metadata และชั้น implementation จำนวนมาก
- `VFS` สำคัญมากในระบบ Unix-like
- allocation method มีผลทั้ง performance และ fragmentation
- free-space management และ recovery เป็นหัวใจของความเสถียร

