# 2. Cache (Memory Organization 1)

## ภาพรวม
บทนี้อธิบายว่าทำไมระบบหน่วยความจำต้องมีลำดับชั้น (memory hierarchy) และ cache ช่วยลดปัญหา CPU เร็วกว่า memory ได้อย่างไร โดยแกนสำคัญคือ `locality`, `cache organization`, `miss`, `write policy` และวิธีที่ software เขียนให้ใช้ cache ได้คุ้มขึ้น

## 1) ปัญหาพื้นฐานของหน่วยความจำ
- ในอดีต memory เร็วพอ ๆ กับ processor แต่ปัจจุบัน processor เร็วกว่า memory มาก
- ถ้า memory access ช้า แม้มีสัดส่วนไม่มาก ก็ทำให้ CPU เสียเวลารอมหาศาล

ตัวอย่างในสไลด์:
- ถ้า memory operation ใช้ 50 cycles
- operation อื่นใช้ 1 cycle
- และมี memory operation 10%
- เวลาที่ CPU ต้องรอ memory จะสูงประมาณ 85%

สรุปคือ ระบบโดยรวมจะช้าลงหนักมากถ้าแก้ memory bottleneck ไม่ได้

## 2) เป้าหมายการออกแบบ
อยากได้หน่วยความจำที่:
- ใหญ่
- เร็ว
- ราคาถูก

แต่ในโลกจริงได้ครบพร้อมกันยาก จึงต้องใช้:
- `Hierarchy`
- `Parallelism`

แนวคิดหลัก:
- หน่วยความจำเล็กมักเร็วแต่แพง
- หน่วยความจำใหญ่กว่ามักช้ากว่าแต่ถูกกว่า
- เลยใช้หลายระดับซ้อนกัน

## 3) Memory Hierarchy
ลำดับชั้นโดยทั่วไป:
- Registers
- Cache
- Main memory (DRAM)
- Secondary storage เช่น SSD/HDD

ระดับที่ใกล้ CPU:
- เร็วกว่า
- เล็กกว่า
- แพงกว่าต่อ byte

การจัดการแต่ละระดับ:
- `Registers <-> Memory` พึ่ง software/compiler มาก
- `Cache <-> Memory` จัดการโดย hardware
- `Memory <-> Disk` ใช้ hardware ร่วมกับ operating system และแนวคิด virtual memory

## 4) เทคโนโลยีหน่วยความจำ
`DRAM`
- ใช้ capacitor
- ความหนาแน่นสูง ราคาถูก กินไฟน้อยกว่า
- ช้ากว่า และต้อง refresh
- เหมาะเป็น main memory

`SRAM`
- ใช้ transistor
- เร็วกว่า
- แพงกว่า ความหนาแน่นต่ำกว่า
- เหมาะเป็น cache

นอกจากนี้ยังมี:
- `Semi-random access` เช่น disk ที่ต้อง seek/rotate
- `Sequential access` เช่น tape

## 5) Principle of Locality
โปรแกรมไม่ได้เข้าถึงทุก address อย่างสุ่มหมด แต่มีแนวโน้มใช้ข้อมูลบางก้อนซ้ำ ๆ

มี 2 แบบ:
- `Temporal locality`
  ถ้าเพิ่งใช้ข้อมูลนี้ ไม่นานมักใช้อีก
- `Spatial locality`
  ถ้าใช้ตำแหน่งหนึ่งแล้ว มักใช้ตำแหน่งใกล้เคียงด้วย

สรุปให้เห็นต่างชัด ๆ:
- `Temporal locality` เน้นว่า "ข้อมูลเดิม" ถูกใช้ซ้ำในเวลาใกล้กัน
- `Spatial locality` เน้นว่า "ข้อมูลข้าง ๆ กัน" มักถูกใช้ต่อเนื่องกัน

ตัวอย่างแบบเห็นภาพ:
- อ่านตัวแปร `sum` ในลูปซ้ำ ๆ = temporal locality
- เดินอ่าน `A[0], A[1], A[2], A[3]` ต่อกัน = spatial locality

ตัวอย่างจริง:
- เพิ่งโทรหาใคร ก็มักโทรหาเขาอีกเร็ว ๆ นี้ = temporal
- โทรหาเพื่อน แล้วอาจโทรหาเพื่อนของเพื่อนต่อ = spatial
- ใช้ดินสอกับยางลบบ่อย ก็ควรพกไว้ใกล้กัน

## 6) คำศัพท์สำคัญของ Cache
- `Hit` หาเจอใน cache
- `Hit rate` โอกาสที่หาเจอ
- `Hit time` เวลาที่เข้าถึง cache
- `Miss` หาไม่เจอ
- `Miss rate = 1 - hit rate`
- `Miss penalty` เวลาที่ต้องไปดึงข้อมูลจากระดับล่างขึ้นมา
- `Block` หรือ `cache line` คือหน่วยข้อมูลที่ย้ายเข้าออก cache ทั้งก้อน

สูตรสำคัญ:
- `Memory Access Time = Hit Time + (Miss Rate x Miss Penalty)`

## 7) คำถาม 4 ข้อของการออกแบบ Cache
ทุกระบบ cache ต้องตอบให้ได้ว่า:
- block ไปวางที่ไหน
- ถ้าข้อมูลอยู่ใน cache จะหาเจออย่างไร
- miss แล้วจะไล่ block ไหนออก
- write จะจัดการอย่างไร

## 8) Direct-Mapped Cache
เป็นแบบง่ายที่สุด

แนวคิด:
- address หนึ่ง block จะถูกบังคับให้ลงได้เพียงตำแหน่งเดียวใน cache
- ใช้ `index` เลือกตำแหน่ง
- ใช้ `tag` ตรวจว่าข้อมูลในตำแหน่งนั้นคือ block ที่ต้องการจริงหรือไม่

ภาพจำ:
- นักศึกษา 100 คนอยู่ในหอประชุม แต่ในห้องอาจารย์มีเก้าอี้ 10 ตัว
- รหัสนักศึกษาบอกได้ว่าต้องนั่งเก้าอี้ตัวไหน
- ป้ายชื่อบนเก้าอี้คือ tag

สูตรในสไลด์:
- `index = (address mod cache size) / block size`
- `tag = address div cache size`

จุดเด่น:
- hit time ต่ำ วงจรง่าย

จุดด้อย:
- conflict miss สูง เพราะตำแหน่งถูกบังคับ

## 9) Valid Bit และ Dirty Bit
- `Valid bit` บอกว่าบรรทัดนี้มีข้อมูลที่ใช้ได้หรือยัง
- `Dirty bit` บอกว่าข้อมูลใน cache ถูกแก้แล้วแต่ยังไม่เขียนกลับ main memory

## 10) Spatial กับ Block Size
เพิ่ม block size จะช่วย spatial locality เพราะดึงข้อมูลข้างเคียงมาด้วย

แต่มีผลเสีย:
- miss penalty สูงขึ้น
- ถ้า cache size คงที่ จะมีจำนวน blocks น้อยลง
- temporal locality อาจแย่ลง เพราะ cache แน่นเร็วขึ้น

ดังนั้น block ใหญ่ไม่ได้แปลว่าดีกว่าเสมอ ต้อง benchmark

## 11) แหล่งที่มาของ Miss
`Compulsory miss` หรือ cold miss
- ครั้งแรกที่เข้าถึงข้อมูลนั้น
- เลี่ยงไม่ได้ทั้งหมด

`Capacity miss`
- cache เล็กเกินไปเมื่อเทียบกับ working set
- ตัวอย่าง ping-pong effect: cache มีน้อยจนสลับเอาข้อมูลเข้าออกตลอด

`Conflict miss`
- ยังมีที่ว่าง แต่ mapping บังคับให้ข้อมูลชน index เดิม
- แก้ด้วยเพิ่ม cache size หรือเพิ่ม associativity

## 12) Cache Configurations หลัก
สิ่งที่มีผลต่อ performance:
- `Cache size`
- `Block size`
- `Associativity`
- `Replacement algorithm`
- `Write management`

## 13) Associativity
`Direct-mapped`
- 1 ที่ต่อ index
- เร็ว แต่ conflict miss สูง

`Set-associative`
- มีหลาย ways ต่อ index เช่น 2-way, 4-way
- conflict miss ลดลง
- แต่ต้องมี comparator หลายตัวและ mux ใหญ่ขึ้น จึง hit time อาจเพิ่ม

`Fully associative`
- ไม่มี index ตายตัว วางได้ทุกที่
- conflict miss = 0
- แต่ฮาร์ดแวร์แพงมาก เพราะต้องเทียบ tag หลายตำแหน่ง

## 14) Replacement Algorithm
ใช้ใน cache ที่มีหลายทางเลือกใน set เดียว

แบบสำคัญ:
- `LRU` ไล่ของที่ใช้นานสุด
- `Round Robin` สลับกันแทนที่
- อื่น ๆ เช่น FIFO, Random

หมายเหตุ:
- direct-mapped แทบไม่มีทางเลือกเรื่อง replacement เพราะตำแหน่งถูกกำหนดอยู่แล้ว

## 15) การลด Miss Penalty
แนวคิดหลักคือทำให้ miss แล้วไปดึงข้อมูลจากระดับที่ยังไม่ไกลเกินไป

ตัวอย่าง:
- มี cache ระดับ 2 (L2) คั่นระหว่าง L1 กับ main memory
- miss จาก L1 อาจ hit ที่ L2 ทำให้โทษของ miss ลดลง

## 16) Write Management
`Write-back`
- เขียนที่ cache ก่อน
- เขียนกลับ main memory ตอนถูกแทนที่
- เร็วสำหรับงานที่เขียนซ้ำ
- แต่ข้อมูลระหว่างทางอาจยังไม่ตรงกับ main memory

`Write-through`
- เขียนทั้ง cache และ main memory ทุกครั้ง
- ง่ายและข้อมูลสอดคล้องกว่า
- แต่ช้ากว่า โดยเฉพาะงานเขียนบ่อย

## 17) Write Allocate vs Write Not Allocate
กรณี write miss:

`Write allocate`
- โหลด block เข้ามาก่อนแล้วค่อยเขียน
- เหมาะเมื่อคาดว่าจะเขียนซ้ำ

`Write not allocate`
- เขียนผ่านไปเลยโดยไม่โหลดเข้ามา
- เหมาะเมื่อไม่อยากเสีย miss penalty เพิ่ม

ในสไลด์เลือกอธิบายแบบง่ายโดยใช้ `write-through + write allocate`

## 18) Write Buffer
ใช้กับ write-through เพื่อลดผลเสียจากการต้องเขียน memory ทุกครั้ง

หลักการ:
- CPU เขียนลง buffer ก่อน
- buffer ค่อยส่งต่อไป main memory

ข้อดี:
- CPU ไม่ต้องรอทุก write

ปัญหา:
- ถ้า memory ช้ามาก buffer อาจเต็ม (`write buffer saturation`)
- วิธีแก้แบบง่ายคือมี L2 cache มารับภาระบางส่วน

## 19) Software กับ Cache
แม้ cache จะโปร่งใสต่อ software แต่โปรแกรมเมอร์ยังเขียนโค้ดให้ใช้ cache ดีขึ้นได้

แนวทาง:
- ใช้ข้อมูลเดิมซ้ำในช่วงเวลาใกล้กัน เพื่อเพิ่ม temporal locality
- เข้าถึงข้อมูลติดกันใน memory เพื่อเพิ่ม spatial locality
- ลดการชนกันของหลาย array ที่ map ไป index เดียวกัน

## 20) ตัวอย่าง Row-major vs Column-major
ภาษา C เก็บ array แบบ `row-major`

ดังนั้น:
- วน `row` ด้านนอก `column` ด้านใน มักเร็วกว่า
- เพราะเข้าถึงข้อมูลติดกันใน memory

ส่วนการเดินแบบ column-major บนข้อมูลที่เก็บแบบ row-major จะกระโดดหน่วยความจำถี่ ทำให้ cache ใช้ spatial locality ได้น้อยลง

## 21) ลด Conflict Miss ด้วยโครงสร้างข้อมูล
บางครั้ง `array of structures` ดีกว่า `separate arrays`

ตัวอย่าง:
- ถ้า `a[i]`, `b[i]`, `c[i]` จากหลาย array ไปชน index เดิมบ่อย
- การรวมเป็น `struct` อาจช่วยให้ข้อมูลที่ใช้ร่วมกันอยู่ใกล้กันและลด conflict miss

## 22) Multilevel Cache
แนวคิดคือ cache หลายชั้นช่วยประนีประนอมระหว่าง hit time, miss rate และ miss penalty

สำหรับ L1:
- `Access Time0 = Hit Time0 + (Miss Rate0 x Miss Penalty0)`
- โดย `Miss Penalty0` อาจเท่ากับเวลาของ L2

## สรุปสั้น
- cache ทำให้รู้สึกว่า memory เร็วขึ้นโดยอาศัย locality
- performance ขึ้นกับ hit time, miss rate, miss penalty
- miss มี 3 แบบ: compulsory, capacity, conflict
- ตัวแปรออกแบบสำคัญคือ size, block size, associativity, replacement, write policy
- software ที่จัดรูปแบบการเข้าถึงข้อมูลดี จะช่วย cache ได้มาก

## Q&A
- ถาม: temporal locality vs spatial locality ต่างกันยังไง?
  ตอบ: temporal locality คือใช้ "ข้อมูลเดิม" ซ้ำในเวลาใกล้กัน ส่วน spatial locality คือใช้ "ข้อมูลที่อยู่ใกล้กันใน memory" ต่อเนื่องกัน เช่น อ่านตัวแปรเดิมซ้ำ ๆ เทียบกับเดินอ่านสมาชิก array ที่ติดกัน
