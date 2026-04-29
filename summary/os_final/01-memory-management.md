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
- `Logical memory` เน้นมุมมองของโปรแกรมว่ามองเห็น address space แบบไหน
- ส่วน `virtual memory` คือกลไกที่ทำให้ logical address space นี้ใหญ่กว่า RAM จริงได้
- การแยกสองมุมมองนี้ทำให้เขียนโปรแกรมง่ายขึ้น และ OS ควบคุมความปลอดภัยได้ดีขึ้น

ตัวอย่าง:
- โปรแกรมอาจคิดว่าตัวเองเริ่มที่ address `0`
- แต่จริง ๆ OS อาจวางข้อมูลของมันไว้คนละตำแหน่งใน RAM

## 3) Address Translation และ MMU
- ทุก logical address ต้องถูกแปลงเป็น physical address ก่อนใช้งานจริง
- ฮาร์ดแวร์ที่ช่วยแปลงคือ `Memory Management Unit (MMU)`
- `MMU` เป็นตัวลงมือแปล address
- ส่วน `page table` เป็นตารางข้อมูลที่บอกว่า page ไหน map ไป frame ไหน
- แนวคิดนี้ทำให้โปรแกรมไม่ต้องรู้ว่าตัวเองถูกวางไว้ตรงไหนใน RAM

## 4) Contiguous Allocation
เป็นวิธีพื้นฐานที่ให้ process แต่ละตัวได้พื้นที่ต่อเนื่องกันก้อนเดียว

ให้นึกภาพเหมือนการจองโต๊ะยาว 1 โต๊ะให้คนกลุ่มหนึ่งนั่งติดกัน:
- ถ้ามีที่ว่างเป็นแถบยาวพอ ก็วางได้ง่าย
- แต่ถ้าที่ว่างกระจายเป็นช่วง ๆ ถึงรวมกันแล้วจะพอ ก็ใช้ไม่ได้ถ้าช่วงนั้นไม่ติดกัน

flow การแปล address:
- OS เก็บแค่ว่า process นี้เริ่มที่ `base` ไหน และยาวได้ถึง `limit` เท่าไร
- CPU ส่ง `logical address` หรือ offset มา
- `MMU` ตรวจว่า offset ยังไม่เกิน `limit`
- ถ้าไม่เกิน ก็เอา `base + offset` ได้เป็น `physical address` ทันที

ตัวอย่างจริง:
- process A ถูกวางไว้ที่ physical address `14000-17999`
- ดังนั้น `base = 14000` และ `limit = 4000`
- ถ้าโปรแกรมอ้าง logical address `120`
- `MMU` ตรวจว่า `120 < 4000` จึงเข้าถึงได้
- physical address = `14000 + 120 = 14120`
- แต่ถ้าโปรแกรมอ้าง logical address `4500` จะเกิน `limit` และเกิด error เพราะออกนอกพื้นที่ของ process

เพราะทั้ง process อยู่ติดกันก้อนเดียว จึงไม่ต้องมี `page table` มาคอยบอกทีละ page ว่าอยู่ frame ไหน

ข้อดี:
- โครงสร้างง่าย
- แปล address ไม่ซับซ้อน

ข้อเสีย:
- เกิด `external fragmentation` ได้ง่าย
- คือมีพื้นที่ว่างรวมกันอาจพอ แต่แตกเป็นหลายช่องเล็ก ๆ จนไม่มีช่องที่ติดกันใหญ่พอสำหรับ process ใหม่
- การหาพื้นที่ว่างก้อนใหญ่พอจะยากขึ้นเมื่อระบบรันไปนาน ๆ
- การขยาย process ระหว่างรันทำได้ไม่สะดวก

แนวทางแก้:
- `compaction` คือย้าย process ให้มาชิดกันเพื่ิอรวมช่องว่างให้เป็นก้อนใหญ่
- แต่การย้ายข้อมูลมี cost สูง
- วิธีที่ใช้กันมากกว่าคือ `paging` เพราะไม่ต้องบังคับให้ memory ของ process อยู่ติดกันทั้งก้อน

## 5) Paging
แนวคิดหลักคือแบ่งหน่วยความจำออกเป็นก้อนขนาดคงที่

- `physical memory` แบ่งเป็น `frames`
- `logical memory` แบ่งเป็น `pages`
- ขนาดของ page/frame เท่ากันและมักเป็นกำลังของ 2
- OS แค่หา free frames ให้พอกับจำนวน pages ของ process
- เวลาแปล logical address จะแยกเป็น 2 ส่วน:
- `page number` คือเลขหน้าที่บอกว่าต้องไปดู entry ไหนใน `page table`
- `page offset` คือระยะภายใน page นั้น ว่าต้องไปตำแหน่งไหนต่อ

ตัวอย่าง:
- ถ้า page size = `1024 bytes`
- logical address = `2500`
- page number = `2500 / 1024 = 2`
- page offset = `2500 mod 1024 = 452`
- แปลว่าให้ไปดูว่า page 2 อยู่ที่ frame ไหน แล้วค่อยเลื่อนไปอีก 452 bytes ใน frame นั้น

ขนาดของแต่ละส่วน:
- ถ้า page size = `2^n` bytes, `page offset` จะมีขนาด `n bits`
- ส่วน `page number` จะใช้บิตที่เหลือของ logical address
- เช่นถ้า logical address ยาว `16 bits` และ page size = `1024 bytes = 2^10`
- จะได้ `page offset = 10 bits`
- และ `page number = 16 - 10 = 6 bits`

ข้อดี:
- ไม่ต้องการพื้นที่ต่อเนื่องทั้งก้อน
- ลดปัญหา `external fragmentation`
- เหมาะกับการจัดการหน่วยความจำสมัยใหม่

ข้อแลกเปลี่ยน:
- ยังมี `internal fragmentation` ได้ใน page สุดท้าย
- เช่นถ้า page size = `4 KB` แต่ process ต้องการเพิ่มอีกแค่ `1 KB` ก็ยังต้องจองทั้ง page ทำให้เหลือว่างใน page นั้นอีก `3 KB`
- ต้องมี `page table` เพื่อใช้แปล address

## 6) Page Table และ TLB
- `page table` เก็บ mapping จาก page number ไป frame number
- ถ้ามองง่าย ๆ `MMU` คือคนอ่านแผนที่ และ `page table` คือแผนที่ที่ถูกอ่าน
- ถ้า page table ใหญ่มาก การเปิดดูทุกครั้งจะช้า
- จึงใช้ `TLB (Translation Look-aside Buffer)` เป็น cache ของ page table entries ที่เพิ่งใช้

ผลลัพธ์:
- `TLB hit` แปล address ได้เร็ว
- `TLB miss` ต้องไปดู page table ใน main memory เพิ่ม

`Effective Access Time (EAT)` คือเวลาเฉลี่ยที่ใช้ต่อการเข้าถึง memory 1 ครั้ง เมื่อคิดรวมทั้งกรณี `TLB hit` และ `TLB miss`

สูตรพื้นฐาน:
- `EAT = hit ratio x hit time + (1 - hit ratio) x miss time`

ถ้าสมมติว่า:
- เวลา access memory 1 ครั้ง = `m`
- `TLB hit` ใช้เวลา = `m` สำหรับอ่านข้อมูลจริง และมีค่า lookup ของ TLB เล็กน้อย
- `TLB miss` ต้องดู `page table` ก่อนแล้วค่อยอ่านข้อมูล จึงประมาณเป็น `2m`

จะได้สูตรแบบย่อที่นิยมใช้:
- `EAT = a x (m + tlb) + (1 - a) x (2m + tlb)`
- โดย `a` คือ `TLB hit ratio`

ตัวอย่าง:
- ถ้า `m = 100 ns`
- `tlb = 20 ns`
- `hit ratio = 0.8`
- `EAT = 0.8 x 120 + 0.2 x 220 = 140 ns`

## 7) Page Table ขนาดใหญ่และการแก้ปัญหา
เมื่อ virtual address space ใหญ่มาก page table จะกิน memory มากตามไปด้วย จึงมีวิธีลด overhead เช่น

- `Hierarchical / multilevel paging`
  แบ่ง page table เป็นหลายระดับ
- เก็บเฉพาะส่วนของ page table ที่จำเป็นจริง
- เหมาะกับ address space ใหญ่ที่ไม่ได้ถูกใช้งานเต็มทุกช่วง

แนวคิด:
- ถ้ามี page table ก้อนเดียวขนาดใหญ่มาก แม้ process จะใช้ address space แค่บางส่วน ก็ยังต้องมีตารางใหญ่ทั้งก้อน
- จึงแยก page table ออกเป็นตารางชั้นบนและตารางชั้นล่าง
- ชั้นบนจะบอกว่าต้องไปดูตารางย่อยอันไหน
- ตารางย่อยจะบอกต่อว่า page นั้นอยู่ frame ไหน

ภาพจำ:
- เหมือนสารบัญหนังสือหลายชั้น
- เปิดสารบัญหลักก่อนว่าเรื่องนี้อยู่บทไหน
- แล้วค่อยไปเปิดสารบัญย่อยของบทนั้นเพื่อหาหน้าที่ต้องการ

ตัวอย่างแบบ 2 ระดับ:
- สมมติ `page number` มี `20 bits`
- แบ่งเป็น `10 bits` แรกสำหรับ `outer page table`
- และ `10 bits` หลังสำหรับ `inner page table`
- จากนั้นค่อยต่อด้วย `page offset`
- เวลาแปล address จะทำเป็น
  1. ใช้ 10 bits แรกเลือก entry ใน `outer page table`
  2. entry นั้นชี้ไป `inner page table` ที่เกี่ยวข้อง
  3. ใช้ 10 bits ถัดมาเลือก page ใน `inner page table`
  4. ได้ frame number แล้วนำไปรวมกับ `page offset`

ข้อดี:
- ประหยัด memory เพราะไม่ต้องสร้างตารางย่อยทุกอันตั้งแต่แรก
- สร้างเฉพาะส่วนของ page table ที่ process ใช้งานจริง

ข้อเสีย:
- ถ้าไม่มี `TLB` จะต้องอ่านหลายขั้นขึ้น ทำให้แปล address ช้าลง
- จึงยิ่งต้องพึ่ง `TLB` ช่วย cache ผลการแปล

## 8) ใจความสำคัญของบทนี้
- OS ต้องทำให้โปรแกรมเห็น memory แบบใช้ง่าย แต่ยังควบคุมของจริงได้
- `MMU + page table` เป็นหัวใจของการแปล address
- `Paging` ชนะ contiguous allocation ในแง่ความยืดหยุ่น
- `TLB` สำคัญมากเพราะช่วยให้การแปล address ไม่กลายเป็นคอขวด

## Q&A
- Q: `Contiguous Allocation` คืออะไร?
  A: คือการจัดสรร memory ให้ process อยู่ในพื้นที่ติดกันก้อนเดียว
- Q: `external fragmentation` คืออะไร?
  A: คือการที่ memory ว่างกระจัดกระจายเป็นหลายช่วง ทำให้ไม่มีช่วงติดกันใหญ่พอแม้ว่าพื้นที่รวมจะยังเหลือ
- Q: `external fragmentation` แก้ยังไง?
  A: แก้ได้ด้วย `compaction` หรือหลีกเลี่ยงปัญหาด้วย `paging`
- Q: `MMU` ต่างกับ `page table` ยังไง?
  A: `MMU` เป็นฮาร์ดแวร์ที่แปลง address ส่วน `page table` เป็นข้อมูลที่ใช้บอกวิธีแปลง
- Q: `internal fragmentation` ใน page สุดท้ายคืออะไร?
  A: คือพื้นที่ว่างที่เหลืออยู่ภายใน page สุดท้าย เพราะ process ใช้ไม่เต็มทั้ง page
- Q: ทำไม `contiguous allocation` ไม่ต้องใช้ `page table`?
  A: เพราะ process อยู่ติดกันก้อนเดียว จึงแค่ใช้ `base + offset` ก็หา physical address ได้เลย
- Q: ตัวอย่างการแปล address ใน `contiguous allocation` เป็นยังไง?
  A: ถ้า `base = 14000` และ logical address = `120` ก็แปลได้เป็น physical address `14120`
- Q: `page number` กับ `page offset` คืออะไร?
  A: `page number` บอกว่าจะดู entry ไหนใน `page table` ส่วน `page offset` บอกตำแหน่งภายใน page นั้น
- Q: ขนาดของ `page number` กับ `page offset` คิดยังไง?
  A: `page offset` ขึ้นกับ page size ถ้า page size = `2^n` ก็ใช้ `n bits` ส่วน `page number` คือบิตที่เหลือของ logical address
- Q: `Effective Access Time (EAT)` คืออะไร?
  A: คือเวลาเฉลี่ยของการเข้าถึง memory เมื่อคิดรวมทั้งกรณี `TLB hit` และ `TLB miss`
- Q: `Hierarchical / multilevel paging` คืออะไร?
  A: คือการแบ่ง page table ออกเป็นหลายระดับ เพื่อไม่ต้องสร้างตารางใหญ่ทั้งก้อนและเก็บเฉพาะส่วนที่ใช้จริง
- Q: `logical memory` ต่างกับ `virtual memory` ยังไง?
  A: `logical memory` คือมุมมอง address space ที่โปรแกรมเห็น ส่วน `virtual memory` คือเทคนิคที่ทำให้ address space นั้นยืดหยุ่นและใหญ่กว่า RAM จริงได้
