# 2. Virtual Memory

## ภาพรวม
บทนี้ขยายจาก paging ไปสู่ `virtual memory` ซึ่งทำให้ process มี address space ได้ใหญ่กว่า RAM จริง โดยเอาเฉพาะหน้าที่ใช้งานอยู่มาไว้ใน memory และเก็บส่วนที่เหลือไว้ใน disk เช่น `swap space`

## 1) แนวคิดของ Virtual Memory
- ให้ process เห็น memory ได้มากกว่าขนาด RAM จริง
- ถ้ามองสั้น ๆ `logical memory` คือ address space ที่โปรแกรมเห็น ส่วน `virtual memory` คือวิธี implement ให้ address space นั้นไม่จำเป็นต้องอยู่ครบใน RAM ตลอดเวลา
- ใช้หลัก `locality` คือโปรแกรมมักใช้ข้อมูลเพียงบางส่วนในช่วงเวลาหนึ่ง
- ส่วนที่ยังไม่ใช้สามารถเก็บใน disk ก่อนได้

ประโยชน์:
- รันโปรแกรมใหญ่กว่าหน่วยความจำจริงได้
- เพิ่มจำนวน process ที่อยู่พร้อมกันได้
- แชร์ code/library ระหว่าง process ได้ง่าย
- สร้าง process ได้เร็วขึ้น

## 2) Demand Paging
ระบบจะโหลด page เข้า RAM “เมื่อจำเป็นจริง”

ข้อดี:
- ลด I/O ที่ไม่จำเป็น
- ใช้ memory น้อยลง
- response time ดีขึ้น

แนวคิดคล้าย cache:
- ถ้าหน้าอยู่ใน RAM ก็ใช้ต่อได้
- ถ้ายังไม่อยู่ จะเกิด `page fault`

เทียบกับ `cache`:
- ทั้งคู่ใช้แนวคิดว่าเก็บของที่น่าจะถูกใช้บ่อยไว้ในหน่วยความจำที่เร็วกว่า
- `cache` ย้ายข้อมูลระหว่าง `cache` กับ `RAM`
- `demand paging` ย้ายข้อมูลระหว่าง `disk` กับ `RAM`
- `cache miss` มักจัดการโดย hardware และเร็วกว่า
- `page fault` ใน demand paging ต้องให้ OS เข้ามาช่วยและช้ากว่ามาก

สรุปภาพรวม:
- `cache` = เร่งความเร็ว
- `demand paging` = ทำให้ใช้ memory ได้ยืดหยุ่นและเหมือนมีมากกว่า RAM จริง

## 3) Page Fault
`Page fault` คือกลไกที่ OS ใช้เมื่อ process อ้างถึง page ที่ยังไม่อยู่ใน physical memory

flow แบบเห็นภาพ:
1. CPU รันคำสั่งแล้วอ้างถึง virtual address ตัวหนึ่ง
2. `MMU` ดู `page table` แล้วพบว่า page นี้ยังไม่มีใน RAM (`present bit` เป็น 0)
3. hardware จึง trap เข้า OS เกิดเป็น `page fault`
4. OS ตรวจว่าการอ้างถึงนี้ถูกต้องไหม
5. ถ้า address ไม่ถูกต้องจริง เช่น เข้าพื้นที่ที่ไม่ได้รับอนุญาต ก็ abort process
6. ถ้า address ถูกต้อง แต่ page แค่อยู่ใน disk:
7. OS หา `free frame` ใน RAM
8. ถ้าไม่มี free frame ต้องเลือก page เก่าออกตาม page replacement algorithm
9. ถ้า page เก่าถูกแก้ไขไว้ อาจต้องเขียนกลับ disk ก่อน
10. OS อ่าน page ที่ต้องการจาก disk เข้ามาใส่ frame ที่เลือกไว้
11. อัปเดต `page table` ว่า page นี้อยู่ใน RAM แล้ว
12. อาจอัปเดต `TLB` ด้วย
13. OS สั่งให้กลับไปรันคำสั่งเดิมใหม่อีกครั้ง
14. รอบนี้ `MMU` เจอ page ใน RAM แล้ว จึงเข้าถึงข้อมูลได้สำเร็จ

จุดสำคัญ:
- `page fault` ไม่ได้แปลว่าโปรแกรมพังเสมอไป
- มันมักเป็นแค่สัญญาณว่า page ที่ต้องใช้ยังไม่ได้ถูกโหลดเข้ามา
- ที่ช้าเพราะมีการสลับไปทำงานใน OS และอาจต้องรอ disk I/O

ถ้าอ้างถึงตำแหน่งที่ไม่ถูกต้องจริง ๆ ระบบต้อง abort process

## 4) Copy-on-Write (COW)
ใช้ตอน `fork()` เพื่อให้ parent กับ child แชร์ page เดิมไปก่อน

- ยังไม่ copy ทันที
- จะ copy ก็ต่อเมื่อฝ่ายใดฝ่ายหนึ่งเขียนข้อมูลลง page นั้น

ข้อดี:
- สร้าง process ได้เร็วขึ้น
- ประหยัด memory
- เหมาะกับ pattern ที่ `fork()` แล้วตามด้วย `exec()`

## 5) Page Replacement
ถ้าเกิด page fault แต่ไม่มี free frame ต้องเลือก page หนึ่งออกจาก memory

อัลกอริทึมสำคัญ:
- `FIFO`
  เอาหน้าเก่าสุดออก แต่มีโอกาสเจอ `Belady's Anomaly`
- `Optimal`
  เอาหน้าที่จะถูกใช้อีกช้าที่สุดออก ดีที่สุดเชิงทฤษฎี แต่ใช้จริงยาก
- `LRU`
  เอาหน้าที่ไม่ได้ใช้มานานที่สุดออก ใช้ “อดีต” ประมาณ “อนาคต”
- `Second-Chance / Clock`
  ปรับ FIFO โดยดู `reference bit` เพิ่ม

ขยายความ `Second-Chance / Clock`:
- แต่ละ frame จะมี `reference bit` เช่น 0 หรือ 1
- ถ้า page นั้นเพิ่งถูกใช้งาน bit มักถูกตั้งเป็น 1
- เวลา OS ต้องเลือก page ออก จะมีตัวชี้หมุนวนเหมือนเข็มนาฬิกา

flow:
1. เข็มชี้ไปที่ frame หนึ่ง
2. ถ้า `reference bit = 0` แปลว่าช่วงหลังไม่ค่อยถูกใช้ ก็เลือก frame นี้ออกได้
3. ถ้า `reference bit = 1` แปลว่าเพิ่งถูกใช้ จึง “ให้โอกาสอีกครั้ง”
4. OS จะรีเซ็ต bit ของ frame นั้นเป็น 0 แล้วหมุนเข็มไป frame ถัดไป
5. วนไปเรื่อย ๆ จนเจอ frame ที่ bit เป็น 0 จึงค่อยเอาออก

ข้อดี:
- ทำงานง่ายกว่า `LRU` แท้
- มักให้ผลดีกว่า `FIFO`

ข้อจำกัด:
- ยังไม่แม่นเท่า `LRU` จริง
- เป็นแค่การประมาณว่าอะไร “เพิ่งถูกใช้”

## 6) Belady's Anomaly
- เกิดกับบาง algorithm เช่น FIFO
- คือเพิ่มจำนวน frames แล้ว page faults กลับเพิ่มขึ้น
- เป็นเหตุผลว่าทำไม algorithm เลือกหน้าออกจึงสำคัญมาก

ทำไมถึงเกิด:
- เพราะ `FIFO` ดูแค่ว่า page ไหน "เข้ามาก่อน" ไม่ได้ดูว่า page นั้นกำลังจะถูกใช้อีกหรือไม่
- พอเพิ่มจำนวน frames ลำดับของ page ที่ค้างอยู่ใน RAM อาจเปลี่ยน
- ลำดับใหม่นี้อาจทำให้ page สำคัญถูกไล่ออกในจังหวะที่แย่กว่าเดิม
- เลยเกิดกรณีแปลกที่ memory มากขึ้น แต่ page fault มากขึ้น

ภาพจำ:
- `FIFO` ไม่ได้มีคุณสมบัติแบบ `stack property`
- คือถ้าเพิ่ม frames แล้ว ชุด page ที่อยู่ใน RAM ไม่จำเป็นต้องครอบของเดิมเสมอ
- จึงไม่มีอะไรรับประกันว่า frames มากขึ้นจะ fault น้อยลง

## 7) LRU Approximation
เพราะ LRU แท้ ๆ ต้องใช้ฮาร์ดแวร์ช่วยและอาจแพง
ระบบจริงจึงมักใช้วิธีประมาณ เช่น

- `reference bit`
- `second-chance`
- `clock algorithm`

แนวคิดคือเก็บข้อมูล “เพิ่งใช้หรือไม่” โดยไม่ต้อง track ทุก access แบบเต็มรูปแบบ

สรุปสั้น ๆ:
- `reference bit`
  เป็นบิตที่บอกว่า page นี้เพิ่งถูกใช้หรือไม่ ถ้าใช้แล้วมักตั้งเป็น 1
- `second-chance`
  เป็น FIFO ที่ถ้าเจอ page ที่ bit เป็น 1 จะยังไม่ไล่ออก แต่ล้าง bit เป็น 0 แล้วข้ามไปก่อน
- `clock algorithm`
  คือวิธี implement `second-chance` แบบใช้ pointer หมุนเป็นวงกลมเหมือนเข็มนาฬิกา

## 8) Memory-Mapped Files และ Shared Memory
`Memory-mapped file` ทำให้ file I/O ถูกมองเหมือนการเข้าถึง memory ปกติ

ทำงานยังไง:
1. process ขอ map ไฟล์เข้ากับช่วงหนึ่งของ virtual memory
2. OS สร้าง mapping ว่า virtual pages ชุดนี้ผูกกับส่วนต่าง ๆ ของไฟล์
3. ตอนแรกข้อมูลอาจยังไม่ถูกโหลดเข้า RAM ทั้งหมด
4. เมื่อโปรแกรมอ่านตำแหน่งใดใน mapping นั้น ถ้า page ยังไม่อยู่ใน RAM จะเกิด `page fault`
5. OS โหลดเฉพาะ page ของไฟล์ส่วนนั้นเข้ามา
6. หลังจากนั้นโปรแกรมเข้าถึงข้อมูลได้เหมือนอ่าน array ใน memory
7. ถ้ามีการเขียน ข้อมูลอาจถูก mark ว่า dirty และค่อย sync กลับไฟล์ภายหลัง

ข้อดีที่เห็นภาพ:
- โค้ดอ่านเขียนไฟล์บางแบบง่ายขึ้น เพราะมองไฟล์เหมือนข้อมูลใน memory
- OS โหลดเฉพาะส่วนที่ถูกแตะจริง ไม่จำเป็นต้องอ่านทั้งไฟล์
- หลาย process map ไฟล์เดียวกันเพื่อแชร์ข้อมูลกันได้

ข้อควรระวัง:
- การแตะข้อมูลครั้งแรกอาจช้าเพราะเกิด page fault
- ถ้าไฟล์ใหญ่มากและ access กระโดดไปมา อาจกระทบ performance ได้

ข้อดี:
- เขียน/อ่านไฟล์ได้ง่ายขึ้น
- ลดความซับซ้อนของ I/O บางกรณี
- หลาย process map ไฟล์เดียวกันร่วมกันได้

`Memory-mapped shared memory` คือการที่หลาย process map พื้นที่ memory ก้อนเดียวกันเข้ามาใน virtual address space ของตัวเอง

ทำงานยังไง:
1. OS สร้าง shared region หนึ่งก้อน
2. หลาย process ขอ map region นี้เข้ามา
3. แต่ละ process อาจเห็นมันอยู่คนละ virtual address ก็ได้
4. แต่เบื้องหลัง map ไปยัง physical pages ชุดเดียวกัน
5. เมื่อ process หนึ่งเขียนข้อมูล อีก process ที่ map region เดียวกันจะอ่านค่าใหม่ได้

จุดสำคัญ:
- เร็วกว่า IPC บางแบบ เพราะไม่ต้อง copy ข้อมูลหลายรอบ
- แต่ต้องระวังเรื่อง synchronization เช่น lock หรือ semaphore เพราะหลาย process อาจเขียนชนกัน

นอกจากนี้ virtual memory ยังรองรับ:
- `shared libraries`
- `shared memory`
- `memory-mapped I/O`

## 9) ใจความสำคัญของบทนี้
- virtual memory ทำให้ระบบยืดหยุ่นกว่า RAM จริงมาก
- `demand paging` ช่วยโหลดเฉพาะส่วนที่ต้องใช้
- `page fault` เป็นกลไกปกติ ไม่ใช่ error เสมอไป
- ประสิทธิภาพขึ้นกับ `page replacement`
- `COW` และ `memory-mapped files` เป็นประโยชน์สำคัญในระบบจริง

## Q&A
- Q: `logical memory` ต่างกับ `virtual memory` ยังไง?
  A: `logical memory` คือมุมมอง address ที่โปรแกรมเห็น ส่วน `virtual memory` คือกลไกที่ทำให้ address space นี้ใหญ่กว่า RAM จริงและโหลดเข้าออกได้ตามต้องการ
- Q: `demand paging` ต่างกับ `cache` ยังไง?
  A: คล้ายกันที่เก็บของที่ต้องใช้ไว้ในที่เร็วกว่า แต่ `cache` ย้ายข้อมูลระหว่าง cache กับ RAM ส่วน `demand paging` ย้าย page ระหว่าง disk กับ RAM และ `page fault` ช้ากว่า `cache miss` มาก
- Q: `page fault` เกิดแล้ว flow เป็นยังไง?
  A: `MMU` เจอว่า page ยังไม่อยู่ใน RAM จึง trap เข้า OS ให้โหลด page จาก disk, อัปเดตตาราง, แล้วกลับมารันคำสั่งเดิมใหม่
- Q: `Second-Chance / Clock` ทำงานยังไง?
  A: มันหมุนดู frame ไปทีละตัว ถ้า `reference bit = 1` จะให้โอกาสอีกครั้งโดยล้างเป็น 0 แล้วข้ามไป จนกว่าจะเจอ frame ที่ bit เป็น 0 จึงเลือกออก
- Q: ทำไม `Belady's Anomaly` ถึงเกิดได้?
  A: เพราะ `FIFO` ดูแค่ว่าใครเข้ามาก่อน ไม่ได้ดูว่าใครกำลังจะถูกใช้อีก ทำให้พอเพิ่ม frames แล้วลำดับ page ในคิวเปลี่ยนและอาจไล่ page สำคัญออกในจังหวะที่แย่กว่าเดิม
- Q: `reference bit`, `second-chance`, `clock algorithm` ต่างกันสั้น ๆ ยังไง?
  A: `reference bit` คือบิตบอกว่า page เพิ่งถูกใช้ไหม, `second-chance` คือการให้ page ที่ bit เป็น 1 รอดไปก่อน, และ `clock algorithm` คือการทำ second-chance ด้วย pointer หมุนเป็นวงกลม
- Q: `Memory-Mapped Files` คืออะไร?
  A: คือการ map ไฟล์เข้ามาใน virtual memory ทำให้โปรแกรมเข้าถึงข้อมูลในไฟล์เหมือนอ่านเขียนข้อมูลใน memory ได้เลย
- Q: `Memory-mapped shared memory` คืออะไร?
  A: คือการ map พื้นที่ memory ก้อนเดียวกันให้หลาย process ใช้ร่วมกัน ทำให้เขียนแล้วอีก process เห็นค่าเดียวกันได้
- Q: `Copy-on-Write (COW)` ช่วยอะไร?
  A: ทำให้ parent กับ child แชร์ page เดิมก่อน แล้วค่อย copy เมื่อมีฝั่งใดฝั่งหนึ่งเขียนข้อมูล จึงประหยัดเวลาและ memory
- Q: shared memory ต้องระวังอะไร?
  A: ต้องระวังการเขียนชนกันของหลาย process จึงมักต้องใช้ lock หรือ semaphore ช่วย synchronization
