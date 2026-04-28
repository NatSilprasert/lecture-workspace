# 6. GPU

## ภาพรวม
บทนี้อธิบายว่า GPU เป็น compute accelerator ที่ออกแบบมาเพื่อ `throughput` มากกว่า `latency` เหมาะกับงานขนานจำนวนมาก เช่น image processing, simulation, machine learning โดยเนื้อหาครอบคลุม parallel programming, Amdahl's law, SIMD/SIMT, การจัด execution, memory hierarchy ของ GPU, การย้ายข้อมูลระหว่าง CPU-GPU และแนวโน้มสถาปัตยกรรมเฉพาะงาน

## 1) Parallel Programming คืออะไร
parallel programming คือการทำให้ algorithm เปิดเผยส่วนที่ทำพร้อมกันได้

มี 2 แบบเด่น:
- `Task parallelism`
  แยกงานใหญ่เป็นหลายงานย่อย
- `Data parallelism`
  ทำ operation เดิมบนข้อมูลหลายตัวพร้อมกัน

เหตุผลที่สำคัญ:
- modern hardware ให้ performance จาก parallelism มากขึ้นเรื่อย ๆ

## 2) ตัวอย่างจริง
สไลด์ยกตัวอย่างงานวิจัยตรวจมะเร็ง:
- จำลองเส้นทางของแสงจำนวนมากในเนื้อเยื่อมนุษย์
- แบบ sequential ใช้ 2.5 ชั่วโมง
- แบบ parallel ใช้น้อยกว่า 2 นาที

บทเรียน:
- ถ้างานแยกเป็นหลายเส้นทางอิสระได้ GPU หรือระบบขนานจะได้เปรียบมาก

## 3) ตัวอย่างตรวจข้อสอบ
ใช้สอนแนวคิดพื้นฐานเรื่อง:
- sequential vs parallel
- load balance
- pipeline

ข้อสรุปสำคัญ:
- ก่อนขนานต้องเข้าใจข้อจำกัดของปัญหา
- ถ้างานแต่ละชิ้นใช้เวลาไม่เท่ากัน อาจต้องจัดสมดุลโหลด
- งานนี้แสดงทั้ง task-based parallelism และ data parallelism ได้ ขึ้นกับวิธีแบ่งงาน

## 4) Pipelining และ Stall
แนวคิดเหมือนที่เรียนในบทก่อน:
- แบ่งงานเป็นหลาย stage
- ให้หลายชิ้นงานไหลพร้อมกัน

ถ้ามี `stall` บาง stage จะว่าง ทำให้ throughput ลดลง

GPU จึงพยายามซ่อน stall ด้วย thread จำนวนมาก

## 5) Amdahl's Law

`Amdahl's Law` (ใจความสำคัญ):
- ความเร็วสูงสุดที่ได้จากการทำขนาน จะถูกจำกัดด้วยส่วนที่ยังต้องรันแบบลำดับ (serial part)
- สูตรที่ใช้บ่อย: `Speedup = 1 / (s + (1-s)/N)`
  - `s` = สัดส่วนงานที่เป็น serial
  - `N` = จำนวนหน่วยประมวลผล

ตัวอย่าง:
- ถ้าโปรแกรมมี serial 10% (`s=0.1`) และใช้ 8 คอร์
- `Speedup = 1 / (0.1 + 0.9/8) = 1 / 0.2125 ≈ 4.7`
- แปลว่าแม้มี 8 คอร์ ก็เร็วขึ้นได้ประมาณ 4.7 เท่า ไม่ถึง 8 เท่า เพราะติดคอขวดที่ส่วน serial

ตัวอย่างในสไลด์:
- ขนานได้ 50% speedup สูงสุดประมาณ 2x
- ขนานได้ 25% speedup สูงสุดประมาณ 1.3x
- ขนานได้ 90% speedup สูงสุดประมาณ 10x

ข้อคิด:
- ไม่ใช่แค่ขนาน "ได้" แต่ต้องขนานส่วนที่กินเวลามากที่สุด

## 6) Heterogeneous Computing
แนวคิดคือใช้ฮาร์ดแวร์ที่เหมาะกับงาน

- CPU เด่นด้าน latency
- GPU เด่นด้าน throughput
- ยังมี DSP, configurable logic, hardware IPs และ accelerator อื่น ๆ

มือถือและระบบสมัยใหม่มักเป็น heterogeneous SOC

## 7) พื้นฐานของ GPU
GPU สร้างมาสำหรับงานที่ขนานได้สูงมาก

สไลด์โยงกับแนวคิด:
- `SIMD`
- `SIMT`

สาระสำคัญ:
- ออกคำสั่งหนึ่งครั้ง
- ให้หลาย lanes ทำงานพร้อมกันกับข้อมูลต่างกัน
- ต้องมี workload ที่ขนานได้มากพอเพื่อรักษา throughput

## 8) Warp/Wavefront และการออกคำสั่ง
การประมวลผลของ GPU มักทำเป็นกลุ่ม thread

- NVIDIA เรียก `warp`
- AMD เรียก `wavefront`

การออกคำสั่ง:
- 1 instruction ถูก issue ให้หลาย lanes
- แต่ละ lane มี register ของตัวเอง

## 9) Fine-Grain Multithreading
GPU ใช้การสลับ thread/warp ระดับฮาร์ดแวร์บ่อยมาก

แนวคิด:
- ถ้า warp หนึ่งติด stall เช่น รอ memory
- GPU จะสลับไปทำ warp อื่นทันที

ข้อดี:
- ซ่อน latency ได้ดี

ข้อเสีย:
- single-thread performance ต่ำ
- ต้องมี register และ state สำหรับหลาย thread
- ต้องมีจำนวน threads มากพอให้สลับ

## 10) CPU vs GPU
`CPU`
- เน้นลด latency ของงานเดี่ยว
- ALU ทรงพลัง
- cache ใหญ่
- control ซับซ้อน
- branch prediction และ data forwarding ดี

`GPU`
- เน้น throughput
- มี ALU จำนวนมาก
- cache เล็กกว่า
- control ง่ายกว่า
- ไม่มี branch prediction ซับซ้อนแบบ CPU
- ต้องพึ่ง massive threading เพื่อซ่อน latency

สรุปสั้น:
- CPU เร็วกับงานลำดับ
- GPU เร็วกับงานขนานมหาศาล

## 11) ใช้ CPU และ GPU ร่วมกัน
งานจริงที่ดีมักใช้ทั้งสองตัว

- ส่วน serial ให้ CPU ทำ
- ส่วน parallel ให้ GPU ทำ

ถ้าเอางาน serial ไปลง GPU:
- จะใช้ทรัพยากรไม่คุ้ม
- เพราะ execution model ของ GPU ต้องการหลาย thread ที่คล้ายกัน

## 12) Branch Divergence
ปัญหาใหญ่ของ GPU

เมื่อ thread ใน warp เข้า `if/else` คนละทาง:
- GPU ต้องรันหลาย path แยกกัน
- แล้วเปิด writeback เฉพาะ thread ที่เกี่ยวข้องในแต่ละ path

ผลคือ:
- utilization ลดลง
- throughput ตก

อธิบายแบบเห็นภาพ:
- สมมติ warp หนึ่งมี 4 threads
- มีเงื่อนไข `if (x > 0)` แล้ว thread 0-1 เข้า `if` แต่ thread 2-3 เข้า `else`
- แทนที่จะรันพร้อมกันทีเดียว GPU ต้องรัน path `if` ก่อนโดยให้ thread 0-1 ทำงาน ส่วน 2-3 รอ
- จากนั้นค่อยรัน path `else` โดยให้ thread 2-3 ทำงาน ส่วน 0-1 รอ
- สุดท้ายเหมือนทุกคนไม่ได้เดินพร้อมกันจริง จึงเสีย throughput

ภาพจำ:
- เหมือนนักเรียน 1 แถวต้องเดินตามครูพร้อมกัน แต่ครึ่งแถวจะไปซ้าย อีกครึ่งจะไปขวา
- ครูเลยต้องพาไปซ้ายก่อน แล้วค่อยย้อนมาพาไปขวา ทำให้ช้ากว่าทุกคนไปทางเดียวกัน

วิธีลด/แก้:
- จัดข้อมูลให้ thread ใน warp เดียวกันมีพฤติกรรมคล้ายกัน
- ลด `if/else` ที่แตกทางมากใน inner loop
- ใช้แนว `branchless` เมื่อเหมาะ เช่นเลือกค่าด้วย arithmetic/select แทน branch
- แยกงานคนละเงื่อนไขไปคนละ kernel ถ้าทำได้
- บางกรณี reorder ข้อมูลก่อนคำนวณ เพื่อให้ warp หนึ่งเจอข้อมูลประเภทเดียวกัน

ตัวอย่างในสไลด์:
- โค้ดที่บางสมาชิกเข้า if บางสมาชิกเข้า else ทำให้จำนวนคำสั่งที่รันจริงมากกว่าที่อยากได้

## 13) Memory ใน CPU และ GPU
สไลด์ทบทวน memory hierarchy ของ CPU และโยงสู่ GPU

สาระสำคัญ:
- CPU พึ่ง cache หนักเพื่อซ่อน latency
- GPU ต้องการ bandwidth สูงมากเพราะมี cores/lanes จำนวนมาก

## 14) DRAM Parallelism
ใน DRAM:
- หนึ่ง row ต่อ array จะ active ได้ทีละ row
- ถ้าอ่านจาก row เดิมต่อเนื่องจะเร็วกว่ากระโดดไป row ใหม่

หลักคิด:
- memory scheduling ที่ดีช่วย latency และ parallelism ได้
- policy ตัวอย่างคือ `first-row-hit, first come, first serve`

อธิบายแบบง่าย:
- DRAM ไม่ได้เป็นก้อนเดียว แต่แบ่งเป็นหลาย `banks` ที่ทำงานซ้อนกันได้ระดับหนึ่ง
- ถ้าคำขอไปคนละ bank ระบบอาจให้หลาย bank เตรียม/อ่านข้อมูลทับเวลากันได้ จึงเรียกว่า `DRAM parallelism`
- แต่ถ้าคำขอหลายตัวไปชน bank เดียวกันหรือสลับคนละ row บ่อย ๆ จะต้องปิด row เดิมแล้วเปิด row ใหม่ ทำให้ช้าลง

บทบาทของ `memory scheduling`:
- ตัว controller จะเลือกว่าจะให้คำขอไหนเข้า DRAM ก่อน
- ถ้าจัดคิวดี จะเพิ่ม `row buffer hit` และกระจายงานข้ามหลาย banks ได้ดี
- จึงช่วยทั้งลด latency และเพิ่ม throughput

ตัวอย่างเห็นภาพ:
- สมมติมี 4 requests
  - R1 ไป bank 0, row A
  - R2 ไป bank 1, row C
  - R3 ไป bank 0, row A
  - R4 ไป bank 0, row B
- ถ้า scheduler เลือก R1 แล้วตามด้วย R3 จะได้ `row hit` เพราะยังใช้ row A เดิม
- ระหว่างนั้น bank 1 อาจเตรียม R2 ไปพร้อมกันได้
- แต่ถ้ารีบสั่ง R4 หลัง R1 ทันที bank 0 ต้องสลับจาก row A ไป row B เร็วเกินไป ทำให้เสียเวลาเพิ่ม

สรุป:
- `DRAM parallelism` = ใช้หลาย banks ให้ทำงานซ้อนกัน
- `memory scheduling` = เรียงคิวคำขอให้ชน row/bank อย่างฉลาด เพื่อให้ระบบอ่านเขียนได้เร็วขึ้น

## 15) GPU DRAM และ Bandwidth
GPU ต้องการ memory bandwidth สูงกว่าปกติมาก

จึงใช้หน่วยความจำที่ออกแบบเพื่อ bandwidth เช่น:
- `GDDR`
- `HBM`

เหตุผล:
- cores จำนวนมากต้องกินข้อมูลพร้อมกัน

## 16) Explicit Memory Management
แนวคิดแบบดั้งเดิมของ GPU programming:
- host (CPU) memory กับ device (GPU) memory แยกกัน
- ข้อมูลต้องอยู่ฝั่ง device ก่อนจึงรัน kernel ได้
- ข้อมูลต้องกลับมาฝั่ง host ถ้าจะให้โค้ดฝั่ง CPU ใช้ต่อ

ดังนั้น programmer ต้องจัดการ:
- จะเอาอะไรไปไว้ที่ไหน
- เมื่อไรควรย้ายข้อมูล
- ลดการ transfer ที่ไม่จำเป็น

## 17) CUDA Execution Model
สไลด์อธิบายภาพรวมของ CUDA:
- kernel หนึ่งตัวถูกรันโดย `grid` ของ threads
- grid แบ่งเป็น blocks
- blocks ช่วยเรื่อง locality และการจัด execution

สรุปแบบง่าย:
- โค้ดเดียวกันถูกนำไปรันบน thread จำนวนมาก
- เหมาะกับงาน data parallel

## 18) CUDA Memory Structure
CUDA device มี memory หลายระดับ เช่น:
- registers
- local/shared memory
- global memory

ใจความสำคัญจากสไลด์:
- memory management เป็นหน้าที่ programmer มากกว่าฝั่ง CPU ปกติ
- การวางข้อมูลถูกระดับมีผลต่อ performance มาก

## 19) CPU-GPU Data Transfer และ DMA
การย้ายข้อมูลมักใช้ `cudaMemcpy()`

เบื้องหลังใช้ `DMA`
- hardware ช่วยย้ายข้อมูลโดยไม่ให้ CPU ต้องคัดลอกทีละคำสั่ง
- มักย้ายผ่าน `PCIe`

`DMA = Direct Memory Access`
- คือกลไกที่ให้อุปกรณ์หรือ controller ย้ายข้อมูลเข้า/ออก memory ได้ตรง ๆ
- CPU แค่สั่งเริ่มงาน กำหนดต้นทาง ปลายทาง และขนาดข้อมูล
- จากนั้น DMA ทำการโอนข้อมูลเอง แล้วค่อยแจ้ง CPU เมื่อเสร็จ

ภาพจำ:
- CPU เหมือนผู้จัดการที่บอกว่า "ย้ายของจากจุด A ไปจุด B"
- DMA คือรถขนของที่ไปขนเอง
- ถ้าไม่มี DMA CPU ต้องมาคัดลอกข้อมูลทีละส่วนเอง ทำให้เสียเวลา

ข้อดี:
- CPU ไปทำอย่างอื่นต่อได้
- transfer มีประสิทธิภาพกว่า

## 20) Virtual Memory และ DMA
คอมพิวเตอร์สมัยใหม่ใช้ virtual memory

แต่ DMA ต้องทำงานกับ `physical addresses`

ตอน `cudaMemcpy()`:
- มีการแปล address และตรวจ page ตอนเริ่ม transfer
- ถ้า OS page-out ข้อมูลระหว่าง transfer อาจมีปัญหา

วิธีแก้:
- ใช้ `pinned memory`
  เพื่อป้องกันไม่ให้ page ถูกย้ายออกระหว่าง DMA

## 21) Multi-Stream
สไลด์ยกตัวอย่าง `cudaStream`

แนวคิด:
- stream คือ queue ของงาน
- หลาย stream ช่วยซ้อน transfer กับ computation ได้
- เช่น ขณะ stream หนึ่งกำลัง copy ข้อมูล อีก stream อาจกำลังรัน kernel

ประโยชน์:
- เพิ่มการใช้ทรัพยากร
- ลดเวลารวมของงาน

## 22) Unified Memory
เป็นแนวคิดให้ CPU และ GPU ใช้ address space ร่วมกันง่ายขึ้น

ข้อดี:
- programmer สะดวกขึ้น
- ไม่ต้องจัดการ copy เองทุกครั้ง

หลักการ:
- คล้าย virtual memory
- ใช้ `on-demand page migration`

อธิบายแบบเห็นภาพ:
- เดิม CPU กับ GPU มักมี memory คนละฝั่ง เวลาใช้ข้อมูลต้อง copy ไปมาเอง
- `Unified Memory` ทำให้โปรแกรมมองเหมือนมี memory ก้อนเดียว
- เวลา CPU หรือ GPU ไปแตะข้อมูล page ไหน ระบบจะย้าย page นั้นไปอยู่ฝั่งที่กำลังใช้งานให้อัตโนมัติ
- ถ้าหน้าหน่วยความจำยังไม่อยู่ฝั่งนั้น ก็เกิดการย้ายแบบ `on-demand` คล้าย page fault ใน virtual memory

ตัวอย่าง:
- CPU เตรียม array ไว้
- พอ GPU จะใช้ข้อมูลนั้น ระบบจะ migrate page ที่จำเป็นไป GPU memory
- ถ้าหลังจากนั้น CPU กลับมาขอใช้ page เดิมอีก ระบบก็อาจย้ายกลับหรือทำให้แชร์ตาม policy ของ runtime/OS

สรุปคือ:
- โปรแกรมเมอร์เขียนง่ายขึ้น
- แต่เบื้องหลังยังมีต้นทุนเรื่อง page migration และการจัดการ locality อยู่

ข้อควรเข้าใจ:
- แม้เขียนโปรแกรมง่ายขึ้น แต่การย้าย page ยังมีต้นทุนอยู่

## 23) ข้อจำกัดทางฟิสิกส์
สไลด์ย้ำว่า trade-off หลายอย่างมาจากข้อจำกัดด้าน:
- memory latency
- bandwidth
- พลังงาน

แนวทางตอบโต้:
- ทำ instruction ซับซ้อนขึ้น
- ใช้ VLIW
- ใช้ accelerator เฉพาะทาง

แต่บางทางเลือกทำให้ compiler จัดการยากขึ้น

## 24) ASIC, TPU, Tensor Core
เมื่อ workload เฉพาะมากขึ้น การใช้ฮาร์ดแวร์เฉพาะทางก็คุ้มขึ้น

`ASIC`
- ออกแบบเพื่อโดเมนเฉพาะ
- เล็กลง กินไฟน้อยลง

`TPU`
- ออกแบบมาเน้นงาน machine learning

`Tensor Core`
- ฮาร์ดแวร์เฉพาะสำหรับ multiply-add ของ matrix
- เหมาะมากกับงานภาพและ deep learning

`Systolic Array`
- รูปแบบการจัดคำนวณที่ไหลข้อมูลผ่านหน่วยประมวลผลเป็นจังหวะ
- เหมาะกับ matrix-heavy workloads

## 25) Logarithmic Number System (LNS)
อีกแนวคิดหนึ่งของ accelerator

- แทนค่าจำนวนจริงด้วย log ของมัน
- การคูณ/หารกลายเป็นการบวก/ลบ
- ประหยัดทรานซิสเตอร์และอาจได้ประสิทธิภาพต่อวัตต์ดี

ข้อเสีย:
- precision แย่กว่า IEEE float
- การบวกจริง ๆ กลับซับซ้อนขึ้น

## 26) แนวโน้ม
สไลด์สรุปว่า:
- CPU และ GPU กำลังใกล้กันมากขึ้น
- การ outsource computation ไป accelerator ง่ายขึ้น
- มี vertical integration มากขึ้น คือ software, hardware และ algorithm ออกแบบร่วมกันมากขึ้น

## สรุปสั้น
- GPU เหมาะกับงาน parallel มาก ๆ และเน้น throughput
- ประสิทธิภาพจริงขึ้นกับการลด stall, ลด branch divergence และจัด memory ให้ดี
- การย้ายข้อมูล CPU-GPU เป็นประเด็นสำคัญพอ ๆ กับการคำนวณ
- ระบบสมัยใหม่กำลังไปสู่ accelerator เฉพาะงานมากขึ้น เช่น TPU และ tensor core

## Q&A
- ถาม: branch divergence คืออะไร ตัวอย่างแบบเห็นภาพ และแก้ยังไง?
  ตอบ: คือกรณี thread ใน warp เดียวกันเจอ branch แล้วไปคนละทาง ทำให้ GPU ต้องรันแต่ละ path แยกกัน เช่นครึ่งหนึ่งเข้า `if` อีกครึ่งเข้า `else`; วิธีลดคือจัดข้อมูลให้ thread ใน warp ทำพฤติกรรมคล้ายกัน, ลด branch ในจุดวิกฤต, ใช้ branchless code หรือแยกงานเป็นคนละ kernel
- ถาม: DRAM parallelism คืออะไร และ memory scheduling มีบทบาทยังไง?
  ตอบ: DRAM parallelism คือการใช้หลาย banks ของ DRAM ให้ทำงานซ้อนกันเพื่อลดการรอ ส่วน memory scheduling คือการเลือกเรียงลำดับคำขอให้ได้ row hit มากขึ้นและกระจายงานข้าม banks อย่างเหมาะสม จึงช่วยเพิ่มทั้ง throughput และลด latency
- ถาม: DMA คืออะไร?
  ตอบ: DMA คือกลไกที่ให้อุปกรณ์ย้ายข้อมูลเข้า/ออก memory ได้โดยตรง โดย CPU แค่ตั้งงานแล้วปล่อยให้ controller โอนข้อมูลเอง จึงลดภาระ CPU และทำให้การย้ายข้อมูลมีประสิทธิภาพขึ้น
- ถาม: Unified Memory คืออะไร และมีหลักการยังไง?
  ตอบ: คือระบบที่ทำให้ CPU กับ GPU มอง memory เหมือนก้อนเดียว โดย runtime จะย้าย page ไปมาระหว่างฝั่ง CPU/GPU แบบ on-demand อัตโนมัติ คล้าย virtual memory ทำให้เขียนโปรแกรมง่ายขึ้น แต่ยังมีต้นทุนจาก page migration อยู่
