# 3. Virtual Memory (Memory Organization 2)

## ภาพรวม
บทนี้อธิบายว่าระบบปฏิบัติการและฮาร์ดแวร์ร่วมกันสร้าง "ภาพลวงตา" ให้โปรแกรมคิดว่ามีหน่วยความจำมากและเป็นส่วนตัวได้อย่างไร ผ่าน `virtual memory`, `paging`, `page table`, `TLB` และความสัมพันธ์กับ cache

## 1) ทำไมต้องมี Virtual Memory
`Virtual Memory` คือเทคนิคที่ทำให้แต่ละโปรแกรมเห็นหน่วยความจำเสมือนของตัวเองเป็นช่วง address ต่อเนื่องขนาดใหญ่ แม้ RAM จริงจะมีจำกัดและกระจัดกระจายอยู่ โดยระบบจะแปลจาก `virtual address` ไป `physical address` อัตโนมัติด้วยฮาร์ดแวร์และ OS

เหตุผลหลักมี 3 เรื่อง

- `ขยายความจุเชิงตรรกะ`
  โปรแกรมที่ต้องการ memory มากกว่าที่เครื่องมีจริงยังพอรันได้ โดยยืมพื้นที่จาก secondary storage
- `Relocation`
  แต่ละ process มี address space ของตัวเอง ทำให้ address เดียวกันในคนละโปรแกรมหมายถึงคนละตำแหน่งจริงได้
- `Protection`
  โปรแกรมหนึ่งไม่ควรเข้าถึง memory ของอีกโปรแกรมโดยตรง

ตัวอย่าง:
- โปรแกรมต้องการ 4GB แต่เครื่องมี RAM 2GB ก็ยังพอทำงานได้บางส่วนผ่าน swap

## 2) Paging Organization
แนวคิดคือแบ่ง address space ออกเป็น block ขนาดเท่ากัน

- ฝั่ง virtual เรียก `page`
- ฝั่ง physical เรียก `page frame`

อธิบายคำว่า block ในบริบทนี้:
- `virtual block` = ก้อนข้อมูลฝั่ง virtual address space ของโปรแกรม (เรียก `page`)
- `physical block` = ก้อนข้อมูลฝั่ง RAM จริงที่ขนาดเท่ากับ page (เรียก `page frame`)
- เวลาแปล address ระบบจะจับคู่ `virtual page` ไปยัง `physical page frame`

เมื่อโปรแกรมอ้าง virtual address:
- hardware จะต้องแปลเป็น physical address ก่อน
- ใช้ข้อมูลจาก `page table`

ถ้า page ยังไม่อยู่ใน physical memory:
- จะเกิด `page fault`
- OS ต้องโหลดจาก disk หรือ swap เข้ามา

## 3) มอง Physical Memory เป็น Cache ของ Disk
แนวคิดของ virtual memory คล้าย cache มาก เพียงแต่:
- upper level คือ physical memory
- lower level คือ disk หรือไฟล์

ลักษณะสำคัญ:
- โหลดเมื่อจำเป็นเท่านั้น (`demand load`)
- replacement จัดการโดย operating system
- miss penalty ใหญ่มาก เพราะต้องเข้าถึง disk

แปลความหมายประโยคจากสไลด์:
- `Physical memory acts as a cache for disk` หมายถึง RAM ไม่ได้เก็บทุก page ของโปรแกรมตลอดเวลา แต่เก็บเฉพาะ page ที่กำลังถูกใช้งานบ่อยหรือเพิ่งถูกเรียก
- `Missing item is loaded on fault only` หมายถึงถ้า CPU อ้างถึง page ที่ยังไม่อยู่ใน RAM จะเกิด `page fault` แล้วค่อยโหลด page นั้นจาก disk เข้ามา (ไม่โหลดทั้งหมดตั้งแต่เริ่ม)
- `Replacement policy is handled by OS` หมายถึงตอน RAM เต็ม ระบบปฏิบัติการเป็นคนตัดสินใจว่าจะเอา page ไหนออก เพื่อเอา page ใหม่เข้า (เช่นแนวคิดใกล้เคียง LRU/Clock)

ตัวอย่างสั้น:
- เปิดโปรแกรมใหญ่ แต่ตอนเริ่มใช้งานจริงอาจใช้แค่บางฟังก์ชัน จึงโหลดเฉพาะ page ส่วนนั้นก่อน
- พอไปกดเมนูที่ไม่เคยใช้มาก่อน ค่อยเกิด page fault แล้วโหลด page เพิ่ม

## 4) คุณสมบัติของ Page
- page มักมีขนาดใหญ่กว่า cache block มาก เช่น 4KB
- page fault มีโทษสูงมาก
- การเขียนแบบ `write-through` แพงเกินไป จึงนิยมแนวคิดแบบ `write-back/page-out`
- การจัดการ fault ต้องอาศัย software เพราะ hardware ไม่รู้เรื่อง filesystem และ swap

## 5) Address Translation
virtual address space ใหญ่กว่า physical address space ได้

การแปล address โดยทั่วไปคือ:
- แยก virtual address เป็น `virtual page number` และ `offset`
- ใช้ page number ไปค้น page table
- ได้ `physical page frame`
- นำ frame มาต่อกับ offset เดิม กลายเป็น physical address

จุดสำคัญ:
- `offset` ภายใน page ไม่เปลี่ยน
- ส่วนที่เปลี่ยนคือเลข page/frame

## 6) Page Table
page table เป็นตาราง mapping จาก virtual page ไป physical frame

โดยทั่วไป page table entry อาจเก็บ:
- physical frame number
- valid/present bit
- access permission เช่น read-only
- สถานะที่เกี่ยวกับการจัดการ page

ข้อควรรู้:
- page table มี `ต่อ process`
- โดยทั่วไป `page table อยู่ใน main memory (RAM)` และ CPU จะมีรีจิสเตอร์พิเศษเก็บตำแหน่งเริ่มต้นของ page table ของ process ปัจจุบัน
- ถ้า address space ใหญ่ page table ก็ใหญ่ตาม

## 7) ปัญหาความเร็วของ Address Translation
ถ้าต้องไปอ่าน page table จาก memory ทุกครั้งก่อนเข้าถึงข้อมูลจริง จะช้ามาก

จึงใช้ `TLB` หรือ `Translation Lookaside Buffer`

## 8) TLB
TLB คือ cache ของผลการแปล address ที่เพิ่งใช้

จุดเด่น:
- access time ใกล้เคียง cache
- เร็วกว่าการไปอ่าน page table ใน main memory มาก

ลักษณะทั่วไป:
- ขนาดเล็ก เช่น 128-256 entries
- มักใช้ fully associative หรือ small set-associative

ขั้นตอนคร่าว ๆ:
- ถ้า `TLB hit` ก็ได้ physical frame เร็ว
- ถ้า `TLB miss` ต้องไปดู page table
- ถ้า page ไม่อยู่ใน memory จริง จึงกลายเป็น page fault

Flow เวลา CPU เรียกข้อมูล (แบบเห็นภาพ):
1. CPU สร้าง `virtual address (VA)` จากคำสั่งที่กำลังรัน
2. ส่ง `virtual page number` ไปหาใน `TLB` ก่อน
3. ถ้า `TLB hit`:
   - ได้ `physical frame number` ทันที
   - นำไปประกอบกับ offset กลายเป็น `physical address (PA)`
   - ไปเช็ก cache/main memory ต่อ
4. ถ้า `TLB miss`:
   - hardware/OS ไปอ่าน `page table` ใน RAM
   - ถ้าเจอ mapping และ page อยู่ใน RAM: เติมข้อมูลนี้เข้า TLB แล้วทำต่อ
   - ถ้าไม่เจอ (page ไม่อยู่ใน RAM): เกิด `page fault` ให้ OS โหลด page จาก disk เข้ามา แล้วอัปเดต page table + TLB ก่อนกลับมารันต่อ

ภาพจำ:
- `TLB` เหมือนโพยสั้นๆ หน้าโต๊ะ
- `Page table` เหมือนแฟ้มใหญ่ในตู้
- ถ้าโพยมีข้อมูล (hit) ตอบเร็วมาก, ถ้าไม่มีต้องไปเปิดแฟ้ม (miss), ถ้าแฟ้มบอกว่ายังไม่มีของ ต้องไปหยิบจากโกดัง (disk)

## 9) TLB กับ Cache
การเข้าถึงข้อมูลจริงต้องผ่านทั้ง translation และ cache

ลำดับแบบตรงไปตรงมาคือ:
- รับ virtual address
- แปลผ่าน TLB/page table
- ได้ physical address
- ค่อยไปเช็ก cache

แต่แบบนี้อาจช้า จึงมีเทคนิค `overlapped cache & TLB access`

## 10) Overlapped Cache & TLB Access
แนวคิด:
- ให้ lookup cache และ TLB ไปพร้อมกันบางส่วน
- ทำได้เมื่อส่วน `index` ของ cache อยู่ในบิตที่ไม่เปลี่ยนจาก VA ไป PA

เหตุผลที่เป็นไปได้:
- offset ภายใน page ไม่เปลี่ยนระหว่างการแปล address

อธิบายแบบง่ายให้เห็นภาพ:
- ปกติถ้าเรียงทีละขั้น ต้อง “แปลที่อยู่ก่อน (TLB)” แล้วค่อย “ไปหาใน cache” ทำให้รอ 2 ช่วงเวลา
- `Overlapped` คือเริ่มสองอย่างพร้อมกัน: ใช้บิต `page offset` จาก virtual address ไปชี้ set ใน cache ทันที ขณะเดียวกันก็ให้ TLB แปล page number
- พอผล TLB ออกมา (ได้ physical page/frame) ค่อยใช้เทียบ tag ของ cache ว่าช่องที่หยิบมาถูกตัวจริงไหม
- ถ้าตรง ก็ส่งข้อมูลได้เลย เร็วกว่าแบบรอทีละขั้น

ภาพจำสั้นๆ:
- เหมือนเรา “เดินไปชั้นหนังสือ” พร้อมกับ “เช็กเลขหมวดจากแคตตาล็อก” ในเวลาเดียวกัน
- ถ้าเลขหมวดตรงกับหนังสือที่หยิบเจอ ก็จบทันที

ตรรกะจากสไลด์:
- ถ้า cache hit และ tag ตรงกับ PA ก็ส่งข้อมูลให้ CPU
- ถ้า cache miss หรือ tag ไม่ตรง แต่ TLB hit ก็ไป memory ต่อด้วย PA
- ถ้า TLB ไม่ hit ก็ต้องแปลแบบปกติ

## 11) เมื่อไหร่ overlap ทำไม่ได้
ถ้า cache ใหญ่เกินเมื่อเทียบกับ page size:
- จำนวนบิตที่ใช้ index cache อาจล้ำออกจาก page offset
- บิตบางตัวอาจเปลี่ยนเมื่อแปล VA เป็น PA
- ทำให้เริ่ม cache lookup ล่วงหน้าอย่างปลอดภัยไม่ได้

ตัวอย่างในสไลด์:
- cache 8KB กับ page 4KB
- cache index ต้องใช้ 13 bits แต่ page offset มี 12 bits

วิธีแก้ที่สไลด์เสนอ:
- ใช้ page ใหญ่ขึ้น เช่น 8KB
- ใช้ 2-way set associative cache
- หรือให้ software รับประกันความสัมพันธ์ของบิตบางตำแหน่ง

## 12) Virtual Memory กับ Cache
คำถามสำคัญคือจะเข้าถึง cache ด้วย virtual address ได้ไหม

ประโยชน์:
- อาจลด latency

ปัญหา:
- shared memory หรือหลาย virtual addresses อาจ map ไป physical address เดียวกัน
- เกิด aliasing/synonym issues ได้

จึงต้องออกแบบอย่างระวัง

## 13) ประเด็นนอกขอบเขตที่สไลด์เอ่ยถึง
- segmentation
- overlay
- inverted page table
- multilevel page table
- virtualized page table ในระบบ virtualization

สิ่งเหล่านี้เป็นวิธีทำ memory management ที่ลึกขึ้นจากพื้นฐานในบทนี้

## 14) มองด้วยคำถาม 4 ข้อแบบเดียวกับ Cache
สไลด์ย้ำว่า cache, TLB, virtual memory เข้าใจได้ด้วยคำถามเดิม

- block/page ไปวางได้ที่ไหน
- หาเจออย่างไร
- miss แล้วแทนที่อะไร
- write จัดการอย่างไร

นี่เป็นกรอบคิดที่ดีมากเวลาเปรียบเทียบระบบหน่วยความจำหลายระดับ

## 15) ประเด็นเรื่อง Process Switch
เมื่อ OS สลับ process ต้องพิจารณา:
- `TLB`
- `Page table`
- `Cache`

เพราะ address space เปลี่ยน ทำให้ข้อมูลแปล address เดิมอาจใช้ไม่ได้หรือใช้ได้เพียงบางส่วน ขึ้นกับ architecture

## สรุปสั้น
- virtual memory ทำให้โปรแกรมเหมือนมี memory มากขึ้น แยกจากกัน และปลอดภัยขึ้น
- paging แบ่ง address เป็น pages และ page frames
- page table ใช้แปล virtual address เป็น physical address
- TLB เป็น cache ของผลการแปล ช่วยลดเวลาแปล address มาก
- การ overlap ระหว่าง cache กับ TLB ช่วยลด latency ได้ แต่ต้องออกแบบให้บิต index อยู่ใน page offset

## Q&A
- ถาม: virtual memory คืออะไร?
  ตอบ: คือระบบที่ทำให้โปรแกรมเห็นหน่วยความจำเสมือนขนาดใหญ่และเป็นส่วนตัว โดยแปล virtual address ไป physical address อัตโนมัติ และใช้ disk ช่วยเมื่อ RAM ไม่พอ
- ถาม: virtual block กับ physical block คืออะไร?
  ตอบ: virtual block คือ `page` ในฝั่ง address เสมือนของโปรแกรม ส่วน physical block คือ `page frame` ใน RAM จริง โดยทั้งสองก้อนมีขนาดเท่ากันและ map หากันผ่าน page table
- ถาม: Physical memory เป็น cache ของ disk หมายความว่าอะไร?
  ตอบ: RAM จะเก็บเฉพาะ page ที่จำเป็นก่อน ถ้า page ที่ต้องใช้ยังไม่อยู่จะเกิด page fault แล้วค่อยโหลดจาก disk และถ้า RAM เต็ม OS จะเลือก page เดิมบางตัวออกเพื่อใส่ page ใหม่
- ถาม: page table อยู่ในไหน?
  ตอบ: โดยปกติอยู่ใน `RAM` (main memory) ของระบบ โดยแต่ละ process มีของตัวเอง และใช้ `TLB` เป็น cache ของผลแปลเพื่อลดการเข้าถึง RAM บ่อยๆ
- ถาม: Overlapped Cache & TLB Access คืออะไร?
  ตอบ: คือการ lookup cache และ TLB พร้อมกันเพื่อลดเวลาเข้าถึงหน่วยความจำ โดยอาศัยบิต `page offset` ที่ไม่เปลี่ยนระหว่าง virtual กับ physical address ทำให้เริ่มหา cache ได้ก่อนรอแปล address เสร็จทั้งหมด
- ถาม: TLB คืออะไร และ flow ตอนเรียกข้อมูลเป็นยังไง?
  ตอบ: TLB คือ cache ของผลแปล virtual page -> physical frame; flow คือ CPU หาใน TLB ก่อน, ถ้า hit ไปต่อได้ทันที, ถ้า miss ต้องอ่าน page table, และถ้า page ไม่อยู่ใน RAM จะเกิด page fault ให้ OS โหลดจาก disk
