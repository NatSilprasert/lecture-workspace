# 7. A New Golden Age for Computer Architecture

## ภาพรวม
บทความของ John L. Hennessy และ David A. Patterson มองว่า Computer Architecture กำลังเข้าสู่ “ยุคทองใหม่” อีกครั้ง แต่รอบนี้เหตุผลต่างจากยุคก่อน เพราะไม่ได้อาศัยแค่การสเกล transistor ตาม `Moore's Law` อย่างเดียวอีกแล้ว สาเหตุหลักคือ performance ของ CPU ทั่วไปเริ่มโตช้าลง, พลังงานและความร้อนกลายเป็นข้อจำกัดจริง, งานสมัยใหม่อย่าง `AI`, `cloud`, `security`, `mobile` และ `IoT` ต้องการสถาปัตยกรรมที่เฉพาะทางมากขึ้น จึงเปิดโอกาสให้เกิดนวัตกรรมใหม่ทั้งที่ระดับ `ISA`, `microarchitecture`, `domain-specific accelerators`, `software stack` และกระบวนการพัฒนาฮาร์ดแวร์

แนวคิดสำคัญของบทความมี 3 แกน:
- ซอฟต์แวร์และการยกระดับ `hardware/software interface` สามารถผลักนวัตกรรมสถาปัตยกรรมได้
- ข้อจำกัดยุคใหม่บังคับให้เน้น `efficiency`, `energy`, `security` และ `cost` มากขึ้น
- ตลาดจริงจะเป็นคนตัดสินว่าสถาปัตยกรรมใดอยู่รอด

## 1) บทเรียนจากประวัติศาสตร์: IBM 360, CISC, RISC, Itanium
บทความเริ่มจากการทบทวนประวัติ เพื่อชี้ว่าหลายแนวคิดในอดีตกลับมาในรูปแบบใหม่เสมอ

- `IBM System/360` ประสบความสำเร็จเพราะสร้าง ISA เดียวให้เครื่องหลายระดับราคาใช้ร่วมกันได้ โดยอาศัย `microprogramming`
- ยุคต่อมามีแนวคิด `CISC` ที่ทำ instruction ให้ซับซ้อนขึ้น เพราะตอนนั้นเชื่อว่าช่วยเขียนโปรแกรมง่ายและลดจำนวนคำสั่ง
- เมื่อ compiler และภาษาระดับสูงดีขึ้น คำถามเปลี่ยนเป็น “compiler ใช้คำสั่งอะไรจริง” จึงเกิดโอกาสของ `RISC`

เหตุผลที่ `RISC` ได้เปรียบ:
- คำสั่งง่ายกว่า จึง execute ได้ตรงใน hardware ไม่ต้องพึ่ง microcode หนัก
- เอา memory เดิมที่ใช้เก็บ microcode ไปใช้เป็น `cache` ได้
- compiler ใช้ register ได้ดีขึ้น
- เหมาะกับการทำ pipeline และ clock ให้เร็ว

สูตร performance ที่บทความย้ำคือ:
`Time/Program = Instructions/Program x Cycles/Instruction x Time/Cycle`

ใจความคือ:
- `CISC` อาจใช้จำนวนคำสั่งน้อยกว่า
- แต่ `CPI` สูงกว่าเยอะ
- ในงานเปรียบเทียบที่ cited ในบทความ CISC ใช้คำสั่งประมาณ 75% ของ RISC แต่ใช้รอบต่อคำสั่งมากกว่า 5-6 เท่า จึงทำให้ RISC เร็วกว่าโดยรวมประมาณ 4 เท่า

ภาพรวมตลาด:
- ช่วงปลายยุค PC `x86` ชนะเพราะ ecosystem ใหญ่, software เยอะ, ราคาถูก
- แต่ยุค `post-PC` เช่น smartphone, embedded, IoT ให้ความสำคัญกับพลังงานและต้นทุนมากกว่า
- บทความระบุว่าปัจจุบัน processor ส่วนใหญ่ในโลกเป็น `RISC` และไม่มี CISC ISA ใหม่เกิดขึ้นมานานแล้ว

กรณี `Itanium`:
- `VLIW/EPIC` เคยถูกคาดหวังว่าจะเป็นก้าวถัดจากทั้ง RISC และ CISC
- แนวคิดคือโยนภาระ scheduling ไปที่ compiler
- แต่ในงานจริงกลับไม่คุ้ม เพราะ general-purpose code มี behavior ที่ทำนายยาก
- จึงกลายเป็นตัวอย่างว่าความสวยงามทางแนวคิดไม่ได้แปลว่าตลาดจะยอมรับ

## 2) End of Moore's Law และ End of Dennard Scaling
ในอดีต performance โตเร็วเพราะ 2 แรงหนุนหลัก:
- `Moore's Law`: transistor ต่อชิปเพิ่มขึ้นเรื่อย ๆ
- `Dennard scaling`: transistor เล็กลงแล้วพลังงานต่อ transistor ก็ลดลง ทำให้ power density ใกล้คงที่

แต่สถานการณ์เปลี่ยนไป:
- `Moore's Law` เริ่มช้าลงตั้งแต่ประมาณปี 2000
- บทความบอกว่าในปี 2018 ความสามารถจริงห่างจากแนวโน้มเดิมราว 15 เท่า
- `Dennard scaling` ชะลอหนักในปี 2007 และแทบหมดลงในปี 2012

ผลที่ตามมา:
- เพิ่ม transistor ได้ แต่ไม่ได้แปลว่าเพิ่ม clock หรือ performance ได้แบบเดิม
- การเพิ่มความเร็วเริ่มแลกด้วยพลังงานและความร้อนมากขึ้น
- เราไม่สามารถหวังให้ hardware รุ่นใหม่ “แรงขึ้นเอง” แบบเดิมได้อีก

## 3) ทำไม ILP อย่างเดียวไม่พอ
ช่วงประมาณ 1986-2002 วิธีหลักที่ CPU ใช้เพิ่ม performance คือ `Instruction-Level Parallelism (ILP)` เช่น:
- deeper pipeline
- superscalar
- out-of-order execution
- speculative execution

แนวทางนี้ช่วยมากในยุคหนึ่ง แต่มีต้นทุนสูงขึ้นเรื่อย ๆ

ตัวอย่างจากบทความ:
- สมมติ core หนึ่งมี pipeline 15 stages และ issue ได้ 4 instructions ต่อ cycle
- จะมี instruction ค้างใน pipeline ได้ถึง 60 คำสั่ง
- ในจำนวนนั้นอาจมี branch ประมาณ 15 ตัว

ปัญหาคือ:
- ถ้า branch prediction ผิด ต้องทิ้งงาน speculative ที่ทำไปแล้ว
- เสียทั้งเวลาและพลังงาน
- ยังต้อง restore state กลับไปก่อน branch อีก

ตัวเลขที่บทความยก:
- ถ้าอยากให้ wasted work จาก branch เหลือเพียง 10% ต้องทำนาย branch ให้ถูกถึง 99.3%
- ซึ่งเป็นระดับที่ยากมากสำหรับโปรแกรมทั่วไป
- Figure 4 แสดงว่าบน Intel Core i7 งาน benchmark หลายตัวมี instruction ที่ “ทำไปแล้วแต่ทิ้ง” เฉลี่ยประมาณ 19%

สรุป:
- ILP ยังสำคัญ แต่การดันมันต่อไปเรื่อย ๆ ไม่คุ้มเหมือนเดิม
- ยิ่งหลัง `Dennard scaling` จบลง ต้นทุนพลังงานยิ่งทำให้แนวทางนี้ตัน

## 4) Multicore, Amdahl's Law และ Dark Silicon
เมื่อดัน ILP ต่อไม่คุ้ม อุตสาหกรรมจึงหันไปเพิ่มจำนวนคอร์ กลายเป็นยุค `multicore`

ข้อดี:
- ถ้างาน parallel ได้ดี จะเพิ่ม throughput ได้

ข้อจำกัดสำคัญคือ `Amdahl's Law`
- speedup ของระบบ parallel ถูกจำกัดโดยสัดส่วนงานที่ยังเป็น `serial`
- สูตรพื้นฐาน: `Speedup = 1 / (s + (1-s)/N)`
  - `s` = สัดส่วนงานที่ serial
  - `N` = จำนวน processors/cores

ตัวอย่าง:
- ถ้า serial 10% และใช้ 8 คอร์
- speedup จะได้ประมาณ 4.7 เท่า ไม่ถึง 8 เท่า

บทความชี้อีกว่า multicore ยังมีปัญหาพลังงาน:
- คอร์ที่เปิดอยู่กินไฟ แม้จะช่วยงานได้ไม่เต็มที่
- ถ้าเพิ่มคอร์มากเกินไป พลังงานและความร้อนจะชนข้อจำกัด `TDP`
- จึงเกิดยุค `dark silicon` คือมีทรานซิสเตอร์/คอร์อยู่บนชิป แต่เปิดใช้งานพร้อมกันไม่ได้ทั้งหมด

ตัวอย่างจาก Figure 5:
- ถ้า serial เพียง 1% บนเครื่อง 64 processors ก็ยังได้ speedup แค่ประมาณ 35 เท่า
- แต่ใช้พลังงานเทียบเท่า 64 processors
- ทำให้มีพลังงานส่วนหนึ่งสูญเปล่าไปมาก

## 5) ประสิทธิภาพไม่ได้มาจากฮาร์ดแวร์อย่างเดียว
บทความใช้ Figure 7 ย้ำแรงมากว่า software optimization มีผลมหาศาล

ตัวอย่าง matrix multiply:
- เริ่มจาก `Python` เป็นฐาน
- เขียนใหม่เป็น `C` เร็วขึ้นประมาณ 47 เท่า
- เพิ่ม `parallel loops` ได้เพิ่มอีกประมาณ 7 เท่า
- ปรับ `memory layout` ให้เหมาะกับ cache ได้เพิ่มอีกประมาณ 20 เท่า
- ใช้ `SIMD` ได้เพิ่มอีกประมาณ 9 เท่า

รวมแล้วเวอร์ชันสุดท้ายเร็วกว่า Python เดิมมากกว่า `62,000x`

ใจความสำคัญ:
- abstraction ที่ดีต่อ productivity อาจแลกด้วย performance สูงมาก
- data layout, locality, vectorization, และ parallelization สำคัญมาก
- compiler และ architecture ที่ช่วยลดช่องว่างนี้จึงเป็นพื้นที่วิจัยสำคัญ

## 6) Domain-Specific Architectures (DSA)
หนึ่งในคำตอบหลักของบทความคือ `DSA` หรือ accelerator ที่ออกแบบเพื่อโดเมนเฉพาะ

ความหมาย:
- เป็น processor ที่ยัง programmable ได้
- แต่ถูกออกแบบให้เหมาะกับงานกลุ่มหนึ่ง เช่น graphics, deep learning, networking
- ต่างจาก `ASIC` ที่มักทำงานเฉพาะแบบตายตัวกว่านั้น

เหตุผลที่ DSA มีประสิทธิภาพและประหยัดพลังงานกว่า CPU ทั่วไป:

- ใช้รูปแบบ parallelism ที่เหมาะกับโดเมน
  - เช่น `SIMD` ซึ่ง efficient กว่า `MIMD` สำหรับหลายงาน
  - บางงานใช้ `VLIW` ได้ดี เพราะ workload มีโครงสร้างชัด

- ใช้ memory hierarchy ได้เหมาะกว่า
  - memory access แพงด้านพลังงานมาก
  - งานบางประเภทมี pattern ชัดจน software/DSL ควบคุมข้อมูลได้ดีกว่า cache ทั่วไป
  - จึงมักใช้ `software-managed memories`

- ใช้ precision ต่ำลงได้
  - CPU ทั่วไปมักใช้ 32/64-bit
  - แต่งาน ML หลายประเภทใช้ 4, 8, 16-bit ก็พอ
  - ทำให้เพิ่ม throughput และลดพลังงานได้มาก

- พึ่ง `Domain-Specific Languages (DSLs)`
  - เช่น `Matlab`, `TensorFlow`, `P4`, `Halide`
  - DSL ช่วยเปิดเผย structure ของงานให้ compiler/map ไปยัง DSA ได้มีประสิทธิภาพ

## 7) ตัวอย่าง DSA: TPU และ Systolic Array
บทความยก `Google TPU v1` เป็นตัวอย่างชัดของ DSA สำหรับ neural network inference

ลักษณะเด่น:
- ใช้ `matrix unit` แบบ `systolic array`
- ทำ `256 x 256 multiply-accumulates` ต่อ clock
- ใช้ความละเอียด `8-bit`
- ควบคุมแบบ `SIMD`
- มี local memory ขนาด 24MB แทนการพึ่ง cache แบบ CPU ปกติ
- การเคลื่อนข้อมูลใน chip ถูกควบคุมชัดเจนผ่าน memory channels

ผลลัพธ์ที่บทความรายงาน:
- เร็วกว่า CPU ทั่วไปประมาณ 29 เท่าใน workload inference ที่ใช้จริงใน data center
- energy efficiency ดีกว่ามากกว่า 80 เท่า

นี่ทำให้เห็นว่า architecture ที่เหมาะกับ workload จริง สามารถชนะ CPU ทั่วไปแบบขาดลอยได้

## 8) Open ISA และ RISC-V
อีกโอกาสใหญ่ที่บทความมองเห็นคือ `open architectures`

แนวคิด:
- ถ้า software มี “Linux แบบ open source”
- ฝั่ง hardware ก็ควรมี `ISA` แบบเปิด เพื่อให้หลายองค์กรช่วยกันสร้าง ecosystem ได้

`RISC-V` คือกรณีสำคัญ

จุดเด่น:
- เป็น ISA แบบ `modular`
- มี base instruction set เล็ก
- แล้วค่อยเพิ่ม extension ตามต้องการ เช่น `M`, `A`, `F/D`, `C`
- ถ้า feature ไหนไม่ต้องใช้ก็ไม่จำเป็นต้องแบกไว้ทั้งหมด

ข้อดีของ RISC-V ตามบทความ:
- instruction น้อยและรูปแบบคำสั่งน้อย ทำให้เรียบง่าย
- verification ง่ายขึ้น
- เป็น clean-slate design ที่หลีกเลี่ยง baggage จากสถาปัตยกรรมรุ่นเก่า
- รองรับ DSA ได้ดี เพราะเผื่อ opcode space ไว้สำหรับ custom accelerators

บทความยังชี้ว่า open architecture ช่วยเรื่อง security ด้วย:
- security ไม่ควรพึ่ง obscurity
- ยิ่งเปิดให้คนช่วยตรวจ ช่วยออกแบบ และช่วยทดสอบได้มาก ยิ่งดี
- การที่ implementation เรียบง่ายทำให้ตรวจสอบ correctness ง่ายขึ้น

## 9) Agile Hardware Development
บทความเสนอว่า hardware ก็กำลังเข้าสู่ยุค `agile` เช่นกัน

แนวคิดคล้าย agile software:
- ทำงานเป็นรอบสั้น
- สร้าง prototype เร็ว
- รับ feedback ไว
- reuse IP และ automation ให้มาก

สิ่งที่ทำให้เป็นไปได้:
- `ECAD` tools ยกระดับ abstraction สูงขึ้น
- มี simulator, FPGA prototype, cloud FPGA, และ flow ที่ช่วยประเมิน area/power/performance เร็วขึ้น
- สามารถ iterate หลายรอบก่อน tape-out จริง

Figure 9 สื่อว่า hardware prototyping มีหลายระดับ:
- software simulation
- FPGA
- layout/`tape-in`
- tape-out จริง

บทความย้ำว่าถ้าชิปไม่ใหญ่เกินไป ต้นทุนการทำ chip prototype ไม่ได้แพงอย่างที่หลายคนคิด และ small chip ก็พอพิสูจน์แนวคิดสถาปัตยกรรมใหม่ได้แล้ว

## 10) Security เป็นโจทย์สถาปัตยกรรมโดยตรง
บทความย้ำว่า security ไม่ควรถูกมองเป็นแค่ปัญหาซอฟต์แวร์

ประเด็นสำคัญ:
- อดีตเคยมีแนวคิดด้าน security ใน architecture เช่น protection rings, capabilities
- แต่หลายอย่างถูกลดบทบาทไป เพราะมองว่า overhead สูงและสภาพแวดล้อมยังไม่ hostile
- โลกปัจจุบันไม่เหมือนเดิม เพราะ cloud, shared hardware, และข้อมูลส่วนตัวมหาศาลทำให้ความเสี่ยงสูงขึ้นมาก

บทเรียนสำคัญ:
- feature ด้าน performance บางอย่าง เช่น speculation อาจเปิดช่องโหว่ใหม่
- ตัวอย่างชัดคือ `Meltdown` และ `Spectre` ที่โจมตีระดับ microarchitecture

ดังนั้น architecture ยุคใหม่ต้องคิดเรื่อง:
- isolation
- verification
- attack surface ของ microarchitecture
- hardware/software co-design เพื่อ security

## 11) ข้อสรุปของบทความ
ผู้เขียนมองว่า “ยุคทองใหม่” จะไม่เหมือนยุคเดิมที่เน้น performance อย่างเดียว แต่จะเน้น:
- `performance`
- `cost`
- `energy efficiency`
- `security`

แรงขับหลักของยุคนี้คือ:
- high-level และ domain-specific software
- DSA/accelerators
- open ISA เช่น `RISC-V`
- agile hardware development
- การทำงานแบบ `vertical integration` ระหว่าง application, compiler, architecture และ implementation technology

บทความจบด้วยมุมมองว่าในทศวรรษต่อไปจะมี “Cambrian explosion” ของสถาปัตยกรรมใหม่จำนวนมาก และตลาดจะเป็นคนคัดเลือกผู้ชนะอีกครั้ง

## สรุปสั้น
- ยุคทองใหม่ของ computer architecture เกิดจากข้อจำกัดใหม่ ไม่ใช่จากการสเกล transistor อย่างเดียว
- ILP และ multicore ยังสำคัญ แต่ไม่พอ ต้องมองเรื่อง efficiency, Amdahl's Law, และพลังงาน
- DSA, DSL, open ISA, RISC-V, agile hardware และ security คือหัวใจของทิศทางใหม่
- การออกแบบที่ดีในยุคนี้ต้องคิดทั้ง hardware และ software ไปพร้อมกัน

## Q&A
- ถาม: ใจความหลักของบทความนี้คืออะไร?
  ตอบ: กำลังเกิดยุคทองใหม่ของคอมพิวเตอร์สถาปัตยกรรม โดยขับเคลื่อนด้วย DSA, open ISA, การพัฒนาฮาร์ดแวร์แบบ agile และการออกแบบที่เน้นพลังงานกับความปลอดภัย
- ถาม: Amdahl's Law คืออะไร พร้อมตัวอย่าง?
  ตอบ: คือกฎที่บอกว่า speedup จาก parallel จะติดเพดานที่ส่วน serial ของโปรแกรม เช่น serial 10% ใช้ 8 คอร์จะได้ speedup ประมาณ 4.7 เท่า ไม่ถึง 8 เท่า
- ถาม: ทำไม RISC กลับมาเด่นอีกครั้ง?
  ตอบ: เพราะยุค post-PC ให้ความสำคัญกับพลังงาน ต้นทุน และความเรียบง่ายมากขึ้น ทำให้ RISC เหมาะกว่ากับ mobile, embedded, IoT และ open ISA อย่าง RISC-V
- ถาม: ทำไม ILP ถึงเพิ่มต่อไปเรื่อย ๆ ไม่คุ้ม?
  ตอบ: เพราะต้องพึ่ง speculation และ control logic ซับซ้อนมากขึ้น ทำให้เสียพลังงานสูง และเมื่อ branch prediction ผิดจะเกิด wasted work จำนวนมาก
- ถาม: DSA เด่นกว่่า CPU ทั่วไปตรงไหน?
  ตอบ: เพราะออกแบบให้เหมาะกับ workload เฉพาะ ใช้ parallelism, memory hierarchy และ precision ได้ตรงงานกว่า จึงได้ performance ต่อวัตต์ดีกว่า
- ถาม: ทำไมเรียกว่า “ยุคทองใหม่” ของสถาปัตยกรรม?
  ตอบ: เพราะข้อจำกัดใหม่เปิดพื้นที่นวัตกรรมกว้างอีกครั้ง.
- ถาม: บทเรียนหลักจาก RISC vs CISC คืออะไร?
  ตอบ: simplicity + compiler/hardware co-design ชนะในหลายบริบท.
- ถาม: Moore's Law ช้าลงกระทบการออกแบบยังไง?
  ตอบ: ต้องพึ่ง efficiency มากกว่าหวัง scaling อัตโนมัติ.
- ถาม: Dennard scaling จบแล้วหมายความว่าอะไร?
  ตอบ: เพิ่มทรานซิสเตอร์ไม่ได้แปลว่าเพิ่ม clock ได้ฟรีเหมือนเดิม.
- ถาม: ทำไม speculative execution มีต้นทุนสูงขึ้น?
  ตอบ: ทำนายพลาดแล้วเสียทั้งพลังงานและงานที่ต้องทิ้ง.
- ถาม: multicore ติดข้อจำกัดหลักจากกฎไหน?
  ตอบ: ติด Amdahl's Law จากส่วน serial ที่เหลืออยู่.
- ถาม: dark silicon คืออะไรแบบสั้นที่สุด?
  ตอบ: มีทรานซิสเตอร์บนชิปแต่เปิดพร้อมกันทั้งหมดไม่ได้.
- ถาม: DSA ได้เปรียบ CPU ทั่วไปตรงไหน?
  ตอบ: จูน datapath/memory ให้เข้ากับโดเมนเฉพาะ.
- ถาม: ทำไม RISC-V ถูกพูดถึงมาก?
  ตอบ: เป็น open, modular ISA ที่เปิดทางนวัตกรรมได้กว้าง.
- ถาม: ความปลอดภัยยุคใหม่ต้องคิดต่างจากเดิมอย่างไร?
  ตอบ: ต้องออกแบบ security ตั้งแต่ architecture ไม่ใช่แก้ทีหลัง.
