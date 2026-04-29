# 2. Virtual Memory

## ภาพรวม
บทนี้ขยายจาก paging ไปสู่ `virtual memory` ซึ่งทำให้ process มี address space ได้ใหญ่กว่า RAM จริง โดยเอาเฉพาะหน้าที่ใช้งานอยู่มาไว้ใน memory และเก็บส่วนที่เหลือไว้ใน disk เช่น `swap space`

## 1) แนวคิดของ Virtual Memory
- ให้ process เห็น memory ได้มากกว่าขนาด RAM จริง
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

## 3) Page Fault
`Page fault` คือกลไกที่ OS ใช้เมื่อ process อ้างถึง page ที่ยังไม่อยู่ใน physical memory

ลำดับโดยย่อ:
1. MMU ตรวจพบว่า page ยังไม่อยู่ใน memory
2. เกิด interrupt ไปที่ OS
3. OS ตรวจว่า access ถูกต้องไหม
4. หา free frame หรือเลือก frame เดิมออก
5. โหลด page ที่ต้องการจาก disk
6. อัปเดต page table
7. เริ่มคำสั่งเดิมใหม่

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

## 6) Belady's Anomaly
- เกิดกับบาง algorithm เช่น FIFO
- คือเพิ่มจำนวน frames แล้ว page faults กลับเพิ่มขึ้น
- เป็นเหตุผลว่าทำไม algorithm เลือกหน้าออกจึงสำคัญมาก

## 7) LRU Approximation
เพราะ LRU แท้ ๆ ต้องใช้ฮาร์ดแวร์ช่วยและอาจแพง
ระบบจริงจึงมักใช้วิธีประมาณ เช่น

- `reference bit`
- `second-chance`
- `clock algorithm`

แนวคิดคือเก็บข้อมูล “เพิ่งใช้หรือไม่” โดยไม่ต้อง track ทุก access แบบเต็มรูปแบบ

## 8) Memory-Mapped Files และ Shared Memory
`Memory-mapped file` ทำให้ file I/O ถูกมองเหมือนการเข้าถึง memory ปกติ

ข้อดี:
- เขียน/อ่านไฟล์ได้ง่ายขึ้น
- ลดความซับซ้อนของ I/O บางกรณี
- หลาย process map ไฟล์เดียวกันร่วมกันได้

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

