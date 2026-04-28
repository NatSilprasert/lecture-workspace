# 1. Contemporary Architecture

## ภาพรวม
บทนี้พูดถึงแนวทางเพิ่มประสิทธิภาพของคอมพิวเตอร์สมัยใหม่ โดยเฉพาะการทำงานแบบขนานหลายระดับ ได้แก่ ระดับข้อมูล (Data-Level Parallelism), ระดับคำสั่ง (Instruction-Level Parallelism: ILP) และระดับเธรด (Thread-Level Parallelism) เป้าหมายคือทำให้มีงานหลายชิ้น "กำลังวิ่งอยู่พร้อมกัน" มากที่สุด

## 1) Flynn's Taxonomy
ใช้แบ่งสถาปัตยกรรมตามจำนวน instruction stream และ data stream

- `SISD` = Single Instruction, Single Data
  เครื่องทั่วไปแบบลำดับเดียว เช่น uniprocessor แบบดั้งเดิม
- `SIMD` = Single Instruction, Multiple Data
  คำสั่งเดียวทำกับข้อมูลหลายตัวพร้อมกัน เช่น vector unit, SSE/AVX, GPU
- `MISD` = Multiple Instruction, Single Data
  เจอน้อย มักใช้กับงานที่เน้น fault tolerance หรือความน่าเชื่อถือ
- `MIMD` = Multiple Instruction, Multiple Data
  หลายหน่วยประมวลผลทำงานคนละคำสั่งกับคนละข้อมูล เช่น multicore, multiprocessor, distributed systems

การมอง MIMD ตามหน่วยความจำ:
- `Shared memory` ทุก core ใช้ main memory ร่วมกัน
- `Distributed memory` แต่ละหน่วยมี memory ของตัวเอง ต้องส่งข้อมูลผ่าน network
- `Hybrid` ผสมสองแบบ

## 2) Parallelism 3 ระดับ
- `Instruction-Level Parallelism (ILP)` ทำหลายคำสั่งจากโปรแกรมเดียวพร้อมกัน
- `Thread-Level Parallelism (TLP)` ทำหลาย instruction streams พร้อมกัน
- `Data-Level Parallelism (DLP)` ทำ operation เดิมกับข้อมูลหลายชิ้นพร้อมกัน

ตัวอย่าง:
- โปรแกรมบวก array จำนวนมาก เหมาะกับ DLP
- CPU ที่ออกหลายคำสั่งใน cycle เดียวคือ ILP
- Web server รับหลาย request พร้อมกันคือ TLP

## 3) Vector Processor และ Vector Unit
แนวคิดคือออกคำสั่งเดียวแล้วให้ไปทำกับข้อมูลเป็นชุด

- เหมาะกับงานที่รูปแบบซ้ำกัน เช่น บวกเวกเตอร์ คูณเมทริกซ์ ประมวลผลภาพ
- ตัวอย่างผลิตภัณฑ์: Intel MMX, SSE, AVX, Apple/IBM Altivec, GPU
- ต่างจาก array processor ตรงที่ vector processor มักประมวลผลข้อมูลหลายตัวต่อเนื่องตามเวลา ส่วน array processor คือหลายตัวพร้อมกันในเชิงพื้นที่

ตัวอย่างจริง:
- สั่ง "ทำบะหมี่ 10 ถ้วย" ครั้งเดียว แทนสั่งทีละถ้วย
- คำสั่ง `C = A + B` ที่ compiler vectorize แล้วจะใช้ SIMD instruction แทน loop ธรรมดา

## 4) Pipelining
แบ่งการทำงานของคำสั่งออกเป็นหลาย stage แล้วให้คำสั่งหลายตัวอยู่คนละ stage พร้อมกัน

- ถ้ามี pipeline 5 stage จะมีได้สูงสุด 5 คำสั่งอยู่ในระบบพร้อมกัน
- โดยประมาณ ILP ของ pipeline ยาว `k` stage คือ `k`
- เวลาโดยรวมของ `n` คำสั่งประมาณ `T(n) = (k + (n-1))Tc`

ข้อดี:
- เพิ่ม throughput

ข้อจำกัด:
- ไม่ได้ทำให้คำสั่งเดี่ยวเร็วขึ้นมาก
- ถ้ามี hazard หรือ stall ประสิทธิภาพจะตก

ตัวอย่างจริง:
- การซื้ออาหาร 5 ขั้นตอน เช่น รับถาด สั่ง จ่าย รับอาหาร เติมเครื่องดื่ม คำสั่งหลายตัวไหลตามกันได้

## 5) Superscalar
เป็นการขยาย pipeline ให้ "ออกคำสั่งได้หลายคำสั่งต่อ cycle"

- ถ้า superscalar degree = 2 และ pipeline 5 stage จะมีได้สูงสุด 10 คำสั่ง in-flight
- โดยประมาณ ILP = `k x degree`
- มักมี `out-of-order execution` คือคำสั่งที่มาทีหลังอาจเสร็จก่อน ถ้า dependency อนุญาต

ปัญหาที่ตามมา:
- `WAR` และ `WAW` hazards
- ต้องมี logic จัดตารางและควบคุมที่ซับซ้อนขึ้น

แนวคิด scheduling:
- `In-order issue / in-order completion` เรียบง่าย แต่หยุ่นน้อย
- `In-order issue / out-of-order completion`
- `Out-of-order issue / out-of-order completion` ยืดหยุ่นมากสุด แต่ฮาร์ดแวร์ซับซ้อน

`Instruction window` คืออะไร:
- คือชุดคำสั่งที่ CPU ดึงเข้ามาเก็บไว้ชั่วคราวเพื่อ "มองล่วงหน้า" ว่ามีคำสั่งไหนพร้อมทำก่อน
- CPU จะเลือกคำสั่งที่ dependency พร้อม และหน่วยคำนวณว่าง มา `issue` ก่อน แม้ลำดับโปรแกรมจริงจะมาก่อน-หลังต่างกัน

สรุปคำสำคัญ:
- `Issue` = ส่งคำสั่งเข้า execution unit เพื่อเริ่มทำงาน
- `Completion` = คำสั่งคำนวณเสร็จได้ผลลัพธ์
- ใน OOO คำสั่งที่ issue ทีหลังอาจ completion ก่อน ถ้าใช้เวลาน้อยกว่าและไม่ติด dependency

มีไว้ทำอะไร:
- ลดเวลารอจากคำสั่งช้า (เช่น load จาก memory)
- เพิ่มการใช้หน่วยคำนวณให้คุ้ม (ไม่ปล่อย ALU ว่าง)
- เพิ่ม throughput/IPC โดยรวมของ CPU

ภาพจำง่าย:
- เหมือนมีงานหลายใบวางบนโต๊ะ (`instruction window`)
- งานที่เอกสารครบทำได้เลยก็หยิบทำก่อน (out-of-order issue)
- งานง่ายอาจเสร็จก่อนงานยาก แม้มาทีหลัง (out-of-order completion)
- แต่ตอน "ประกาศผลอย่างเป็นทางการ" CPU มัก commit ตามลำดับเดิมเพื่อให้ผลลัพธ์ถูกต้อง

flow ตัวอย่าง (เห็นภาพเร็ว):
- คำสั่งในโปรแกรม:
  1) `I1: R1 = Mem[A]` (load ช้า)
  2) `I2: R2 = R3 + R4` (ไม่ขึ้นกับ I1)
  3) `I3: R5 = R1 + 1` (ต้องรอ I1)
  4) `I4: R6 = R7 + R8` (ไม่ขึ้นกับ I1)
- แบบ in-order:
  - I1 ไปก่อน แล้วทุกอย่างหลังมันมักรอ
  - I2/I4 ที่จริงทำได้ก็ยังเสียจังหวะ
- แบบ out-of-order + instruction window:
  - ดึง I1..I4 เข้ามาดูพร้อมกัน
  - I1 เริ่มโหลด memory (ยังไม่เสร็จ)
  - เห็นว่า I2 และ I4 พร้อม จึง issue ไปทำก่อน
  - I2/I4 completion ก่อน I1 ได้
  - พอ I1 เสร็จแล้ว I3 จึงทำต่อได้
  - สุดท้าย commit ผลตามลำดับโปรแกรมเพื่อรักษาความถูกต้อง

ตัวอย่างจริง:
- เหมือนมีผู้จัดการแยกคนเข้าคิวข้าวกับคิวก๋วยเตี๋ยว ทำให้คนบางคนเสร็จเร็วกว่าแม้มาทีหลัง

## 6) Superpipeline
ทำให้ 1 cycle ใหญ่ถูกแบ่งเป็นหลาย minor cycles

- ถ้ามี `m` minor cycles ใน pipeline `k` stages จะได้ ILP ประมาณ `mk`
- ช่วยเพิ่ม throughput โดยซอยเวลาให้ละเอียดขึ้น
- ถ้ารวมกับ superscalar ก็ยิ่งเพิ่มจำนวนงานที่ค้างในระบบได้อีก

## 7) VLIW
`Very Long Instruction Word` คือให้ compiler รวมหลาย operation ที่เป็นอิสระไว้ใน instruction word เดียว

- เป็น `static scheduling`
- ฮาร์ดแวร์ง่ายกว่า superscalar เพราะ compiler ช่วยตัดสินใจล่วงหน้า
- แต่ผูกกับรุ่นเครื่องมาก ถ้าสถาปัตยกรรมเปลี่ยน binary compatibility จะลำบาก
- ตัวอย่างสำคัญในอดีตคือ Intel Itanium

ข้อสังเกต:
- ที่ degree เท่ากัน VLIW, superscalar, superpipeline ให้ศักยภาพ performance ใกล้กันในเชิงแนวคิด
- แต่ VLIW พึ่ง compiler มาก และใช้งานจริงยาก

## 8) Performance มุมมองรวม
ตารางในสไลด์ชี้ว่าเมื่อเทียบกับ single-cycle processor:

- multiple-cycle ช่วยได้เล็กน้อย
- pipeline ช่วยได้มากขึ้น
- superscalar, superpipeline และ VLIW ช่วยเพิ่ม speedup ตาม degree ที่ขยาย

แต่ performance จริงไม่ถึงค่าทฤษฎีเสมอ เพราะ:
- latency ของแต่ละ operation ไม่เท่ากัน
- มี data hazard, control hazard, structural hazard
- บางช่วงของโค้ด parallelize ไม่ได้

## 9) Compiler กับ ILP
compiler มีบทบาทมากในการเปิดโอกาสให้ฮาร์ดแวร์ทำงานขนานได้ดีขึ้น โดยเฉพาะเมื่อ basic block เล็กเกินไป

เทคนิคสำคัญ:
- `Register renaming`
  ลด false dependency จากการใช้ register ซ้ำ
- `Scheduling loads`
  ย้ายคำสั่งอิสระมาคั่นหลัง load เพื่อซ่อน latency ของ memory
- `Loop unrolling`
  ขยายหลาย iteration มารวมกัน เพื่อลด overhead ของ branch และเพิ่มช่องให้ reschedule
- `Software pipelining`
  เลื่อนงานจากหลาย iteration มาซ้อนกันอย่างเป็นระบบ โดยไม่ต้องขยายโค้ดมากเท่า unroll

## 10) Loop Unrolling
หลักการคือทำหลาย iteration ต่อหนึ่งรอบ loop

ข้อดี:
- ลด overhead ของ `branch` และ `loop control`
- เปิดโอกาสให้ compiler สลับลำดับคำสั่งข้าม iteration
- ใช้ร่วมกับ register renaming ได้ดี

ข้อเสีย:
- code size โตขึ้น
- ถ้าขยายมากเกินไปอาจไม่คุ้ม
- ถูกจำกัดด้วยจำนวน register และ resource อื่น

จากตัวอย่างในสไลด์:
- loop ปกติใช้ 8 cycles ต่อ iteration
- unroll + reschedule 4 iterations เหลือ 14 cycles ต่อ 4 iterations หรือ 3.5 cycles ต่อ iteration
- ถ้ารวมกับ superscalar ยิ่งลดลงได้อีก

## 11) Software Pipelining
เป็นการจัดตารางข้าม iteration แบบเป็นระบบ โดยแบ่งเป็น 3 ส่วน

- `Start-up` เติม pipeline
- `Kernel` ช่วง steady state
- `Clean-up` ระบายงานค้าง

ข้อดี:
- performance ดี
- code size มักเหมาะสมกว่า unroll หนัก ๆ

## 12) Dynamic vs Static Scheduling
`Dynamic scheduling`
- ตัดสินใจโดยฮาร์ดแวร์ตอนรันจริง
- ช่วยให้โปรแกรมเก่าได้ประโยชน์บนเครื่องใหม่
- รองรับ out-of-order ได้ดี
- แต่ฮาร์ดแวร์ซับซ้อนและกินพลังงานมากขึ้น

`Static scheduling`
- ตัดสินใจโดย compiler
- มองภาพโค้ดได้กว้าง
- ฮาร์ดแวร์ง่ายกว่า
- แต่ dependency บางอย่างรู้ไม่ได้ตอน compile
- compatibility มักแย่กว่า

## 13) Multicore และ Thread-Level Parallelism
เมื่อ ILP จากโปรแกรมเดียวไม่พอ ก็ใช้หลาย thread

- multicore คือมีหลาย core อยู่บน chip เดียว
- แต่ละ core อาจเป็น superscalar อยู่แล้ว
- `Simultaneous Multithreading (SMT)` คือดึงคำสั่งจากหลาย thread เข้ามาเติม pipeline พร้อมกัน
- Intel เรียก implementation นี้ว่า `Hyper-Threading`

## 14) LLVM
LLVM เป็นโครงสร้างเครื่องมือ compiler ที่ช่วยทำ optimization และรองรับหลายภาษา/หลาย architecture ผ่าน IR กลาง

## สรุปสั้น
- สถาปัตยกรรมสมัยใหม่เพิ่มความเร็วด้วยการทำงานขนานหลายชั้น
- SIMD เน้นข้อมูลจำนวนมากแบบรูปแบบเดียวกัน
- pipeline, superscalar, superpipeline, VLIW เน้น ILP
- compiler และ hardware ต้องช่วยกันลด stall และใช้ resource ให้เต็ม
- เมื่อ ILP ไม่พอ จึงขยับไปใช้ TLP ผ่าน multicore และ SMT

## Q&A
- ถาม: instruction window ใน out-of-order issue/out-of-order completion คืออะไร มีไว้ทำอะไร?
  ตอบ: คือพื้นที่ที่ CPU เก็บคำสั่งหลายตัวเพื่อเลือกคำสั่งที่พร้อมทำก่อน ช่วยหลบการรอ dependency/latency, ใช้ execution units ได้เต็มขึ้น และเพิ่ม throughput โดยรวม
- ถาม: ขอตัวอย่าง flow ของ out-of-order issue/out-of-order completion
  ตอบ: ถ้า `load` ตัวแรกช้า CPU จะไม่รอเฉยๆ แต่หยิบคำสั่งคำนวณที่ไม่ขึ้นกับ load มาทำก่อนให้เสร็จได้ แล้วค่อยกลับมาทำคำสั่งที่ต้องพึ่งผล load เมื่อข้อมูลมา

