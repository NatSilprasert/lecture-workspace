# 1. Memory Management

## ภาพรวม
บทนี้อธิบายงานของระบบปฏิบัติการในการจัดการหน่วยความจำ โดยแยกให้ชัดระหว่าง `physical memory` ที่มีจริงในเครื่อง กับ `logical memory` ที่เป็นมุมมองต่อเนื่องที่โปรแกรมเห็น เป้าหมายคือทำให้โปรแกรมใช้งานหน่วยความจำได้ง่าย ปลอดภัย และมีประสิทธิภาพ

## 1) งานหลักของ Memory Management
- ให้โปรแกรมเห็น `logical address space` ที่ใช้งานง่าย
- แปลง logical address ไปเป็น physical address
- ป้องกัน process ไม่ให้รบกวนกัน
- รองรับ memory sharing และ memory-mapped I/O
- วางพื้นฐานให้ `virtual memory` ทำงานได้

## 2) Physical vs Logical Memory
- `Physical memory` คือ RAM จริงที่ทุกโปรแกรมใช้ร่วมกัน
- `Logical memory` คือภาพนามธรรมที่แต่ละ process เห็นเหมือนมีพื้นที่ต่อเนื่องของตัวเอง
- การแยกสองมุมมองนี้ทำให้เขียนโปรแกรมง่ายขึ้น และ OS ควบคุมความปลอดภัยได้ดีขึ้น

ตัวอย่าง:
- โปรแกรมอาจคิดว่าตัวเองเริ่มที่ address `0`
- แต่จริง ๆ OS อาจวางข้อมูลของมันไว้คนละตำแหน่งใน RAM

## 3) Address Translation และ MMU
- ทุก logical address ต้องถูกแปลงเป็น physical address ก่อนใช้งานจริง
- ฮาร์ดแวร์ที่ช่วยแปลงคือ `Memory Management Unit (MMU)`
- แนวคิดนี้ทำให้โปรแกรมไม่ต้องรู้ว่าตัวเองถูกวางไว้ตรงไหนใน RAM

## 4) Contiguous Allocation
เป็นวิธีพื้นฐานที่ให้ process แต่ละตัวได้พื้นที่ต่อเนื่องกันก้อนเดียว

ข้อดี:
- โครงสร้างง่าย
- แปล address ไม่ซับซ้อน

ข้อเสีย:
- เกิด `external fragmentation` ได้ง่าย
- การหาพื้นที่ว่างก้อนใหญ่พอจะยากขึ้นเมื่อระบบรันไปนาน ๆ
- การขยาย process ระหว่างรันทำได้ไม่สะดวก

## 5) Paging
แนวคิดหลักคือแบ่งหน่วยความจำออกเป็นก้อนขนาดคงที่

- `physical memory` แบ่งเป็น `frames`
- `logical memory` แบ่งเป็น `pages`
- ขนาดของ page/frame เท่ากันและมักเป็นกำลังของ 2
- OS แค่หา free frames ให้พอกับจำนวน pages ของ process

ข้อดี:
- ไม่ต้องการพื้นที่ต่อเนื่องทั้งก้อน
- ลดปัญหา `external fragmentation`
- เหมาะกับการจัดการหน่วยความจำสมัยใหม่

ข้อแลกเปลี่ยน:
- ยังมี `internal fragmentation` ได้ใน page สุดท้าย
- ต้องมี `page table` เพื่อใช้แปล address

## 6) Page Table และ TLB
- `page table` เก็บ mapping จาก page number ไป frame number
- ถ้า page table ใหญ่มาก การเปิดดูทุกครั้งจะช้า
- จึงใช้ `TLB (Translation Look-aside Buffer)` เป็น cache ของ page table entries ที่เพิ่งใช้

ผลลัพธ์:
- `TLB hit` แปล address ได้เร็ว
- `TLB miss` ต้องไปดู page table ใน main memory เพิ่ม

## 7) Page Table ขนาดใหญ่และการแก้ปัญหา
เมื่อ virtual address space ใหญ่มาก page table จะกิน memory มากตามไปด้วย จึงมีวิธีลด overhead เช่น

- `Hierarchical / multilevel paging`
  แบ่ง page table เป็นหลายระดับ
- เก็บเฉพาะส่วนของ page table ที่จำเป็นจริง
- เหมาะกับ address space ใหญ่ที่ไม่ได้ถูกใช้งานเต็มทุกช่วง

## 8) ใจความสำคัญของบทนี้
- OS ต้องทำให้โปรแกรมเห็น memory แบบใช้ง่าย แต่ยังควบคุมของจริงได้
- `MMU + page table` เป็นหัวใจของการแปล address
- `Paging` ชนะ contiguous allocation ในแง่ความยืดหยุ่น
- `TLB` สำคัญมากเพราะช่วยให้การแปล address ไม่กลายเป็นคอขวด

