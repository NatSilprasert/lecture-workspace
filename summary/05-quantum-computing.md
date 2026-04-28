# 5. Quantum Computing

## ภาพรวม
บทนี้ปูพื้นฐานคอมพิวเตอร์ควอนตัมตั้งแต่ `qubit`, `Dirac notation`, `measurement`, `Bloch sphere`, วงจรควอนตัม, `entanglement`, `teleportation`, `superdense coding` ไปจนถึงอัลกอริทึมสำคัญ เช่น `Deutsch-Jozsa`, `Grover`, `QFT`, `QPE`, และ `Shor`

## 1) จาก bit ไปสู่ qubit
bit แบบคลาสสิกมีค่า 0 หรือ 1

แต่ `qubit` อยู่ในสถานะซ้อนทับ (`superposition`) ได้

เช่น:
- `|psi> = alpha|0> + beta|1>`
- โดย `|alpha|^2 + |beta|^2 = 1`

ประโยชน์:
- เหมือนระบบถือความเป็นไปได้หลายสถานะพร้อมกัน

อธิบายแบบเห็นภาพ:
- ถ้า bit คลาสสิกคือเหรียญที่วางหงายหัวหรือก้อยอย่างใดอย่างหนึ่ง
- `qubit` คือเหรียญที่กำลังหมุนอยู่ ยังไม่ตกเป็นหัวหรือก้อยแน่ชัด
- ตอนยังไม่วัด เราจะบอกได้แค่ว่ามีโอกาสเป็น `0` และ `1` เท่าไร
- พอ `measure` แล้วจึงได้ผลลัพธ์เป็นอย่างใดอย่างหนึ่ง

ตัวอย่าง:
- ถ้า `alpha = beta = 1/sqrt(2)` จะเขียนเป็น `(|0> + |1>) / sqrt(2)`
- แปลว่าโอกาสวัดได้ `0` กับ `1` เท่ากัน

`Interference` คืออะไร:
- คือการที่ amplitude ของสถานะควอนตัมมารวมกันแล้วเสริมกันหรือหักล้างกัน
- ถ้า phase ไปทางเดียวกัน จะเกิด `constructive interference` ทำให้คำตอบบางตัวเด่นขึ้น
- ถ้า phase ตรงข้ามกัน จะเกิด `destructive interference` ทำให้คำตอบที่ไม่ต้องการหายไปบางส่วน

ภาพจำ:
- เหมือนคลื่นน้ำสองลูก
- ถ้าสันคลื่นตรงกัน คลื่นสูงขึ้น
- ถ้าสันคลื่นชนท้องคลื่น คลื่นหักล้างกัน

ทำไมสำคัญ:
- quantum algorithm ใช้ interference เพื่อเพิ่มโอกาสของคำตอบที่ถูก
- และลดโอกาสของคำตอบที่ไม่ต้องการ ก่อนวัดผล

ข้อจำกัด:
- เมื่อวัด (`measure`) สถานะจะยุบ (`collapse`) ไปเป็นผลลัพธ์ใดผลลัพธ์หนึ่ง
- จึงต้องออกแบบ algorithm ให้ใช้ `interference` ช่วยขยายคำตอบที่ถูกและหักล้างคำตอบที่ไม่ต้องการ

## 2) Dirac Notation และ Density Matrix
สัญกรณ์หลัก:
- `ket` เช่น `|a>`
- `bra` เช่น `<b|`
- `bra-ket` คือ inner product
- `ket-bra` คือ matrix

สถานะพื้นฐาน:
- `|0>`
- `|1>`

ทั้งสองสถานะตั้งฉากกัน (orthogonal)

`Density matrix` ใช้แทนสถานะควอนตัมได้ทุกแบบ

ประเภท:
- `Pure state`
  เกิดจากสถานะเดี่ยวชัดเจน เช่น `rho = |psi><psi|`
- `Mixed state`
  เป็นการผสมเชิงความน่าจะเป็นของหลายสถานะ

## 3) Measurement
การวัดต้องเลือก basis ก่อน

ฐานที่พบบ่อย:
- `Z-basis`: `|0>`, `|1>`
- `X-basis`: `|+>`, `|->`
- `Y-basis`: `|+i>`, `|-i>`

กฎสำคัญคือ `Born rule`
- ความน่าจะเป็นที่จะวัดได้สถานะหนึ่ง เท่ากับขนาดกำลังสองของ amplitude ที่สอดคล้องกัน

ตัวอย่าง:
- ถ้าสถานะให้ amplitude ของ `|1>` มากกว่า ก็มีโอกาสวัดได้ `1` มากกว่า

## 4) Bloch Sphere
pure state ของ qubit หนึ่งตัวสามารถแทนเป็นจุดบนผิวทรงกลม Bloch ได้

ประโยชน์:
- ช่วยมองภาพการหมุนสถานะและ quantum gate ได้ง่ายขึ้น

ข้อควรระวัง:
- มุมบน Bloch sphere กับมุมใน Hilbert space ไม่เท่ากันตรง ๆ

## 5) Quantum Gates: Single-Qubit
วงจรควอนตัมประกอบจาก `gates` ซึ่งแทนด้วย unitary matrices

gate สำคัญ:
- `H` Hadamard
  สร้าง superposition
- `X`
  bit flip
- `Y`
  bit และ phase flip
- `Z`
  phase flip

ภาพจำ:
- gate คือคำสั่งเปลี่ยนสถานะของ qubit อย่างย้อนกลับได้

## 6) หลาย Qubits และ Tensor Product
เมื่อมีหลาย qubits จะใช้ `tensor product` แทนสถานะรวม

ตัวอย่าง:
- `|1> tensor |0>`

ถ้าสถานะเขียนแยกเป็นผลคูณของแต่ละส่วนได้ เรียกว่าไม่ correlated มาก
แต่ถ้าเขียนแยกไม่ได้ อาจเป็น `entangled`

## 7) Two-Qubit Gates และ CNOT
เพราะ quantum evolution ต้องย้อนกลับได้ จึงใช้ reversible gates

gate สำคัญมาก:
- `CNOT`

หน้าที่:
- ถ้า control qubit เป็น 1 จะกลับค่าของ target qubit

เป็น gate หลักที่ใช้สร้าง entanglement และเป็นฐานของวงจรควอนตัมจำนวนมาก

## 8) Entanglement
ถ้าสถานะสองระบบเขียนเป็นผลคูณของสถานะแต่ละระบบไม่ได้ เรียกว่า `entangled`

Bell states 4 แบบ:
- `|psi00> = (|00> + |11>) / sqrt(2)`
- `|psi01> = (|01> + |10>) / sqrt(2)`
- `|psi10> = (|00> - |11>) / sqrt(2)`
- `|psi11> = (|01> - |10>) / sqrt(2)`

Bell states เป็นตัวอย่างของความสัมพันธ์ที่แรงมากกว่าระบบคลาสสิก

การสร้าง Bell state:
- เริ่มจาก basis state
- ใช้ `H` กับ qubit แรก
- แล้วตามด้วย `CNOT`

## 9) Quantum Teleportation
เป้าหมาย:
- Alice ต้องการส่ง qubit ที่ไม่รู้สถานะให้ Bob
- แต่ส่ง classical bits ได้เพียง 2 bits
- ทั้งคู่แชร์ entangled pair ไว้ก่อน

หลักการ:
- Alice ทำการวัดบางส่วน
- ส่งผลการวัด 2 classical bits ให้ Bob
- Bob ใช้ผลนั้นเลือกทำ `I`, `X`, `Z`, หรือ `ZX`
- สถานะเดิมจะไปปรากฏที่ฝั่ง Bob

ประเด็นสำคัญ:
- Alice ไม่ได้ copy สถานะ แต่ส่งต่อโดยทำลายสถานะเดิม
- สอดคล้องกับ `no-cloning theorem`

## 10) Superdense Coding
เป็นแนวคิดกลับด้านกับ teleportation

- ส่ง `2 classical bits`
- โดยใช้การส่ง `1 qubit`
- และต้องมี entangled pair ร่วมกันก่อน

4 ขั้น:
- preparation
- encoding
- transmission
- decoding

แนวคิด:
- Alice เลือก gate ตามข้อความ 2 บิตที่ต้องการส่ง
- ส่ง qubit ไปให้ Bob
- Bob decode ด้วย CNOT + H แล้ววัดออกมา

## 11) Quantum Programming และ Noise
สไลด์ยกตัวอย่างใช้ `Qiskit`

ประเด็นสำคัญ:
- transpiler สามารถ optimize circuit ให้ gate count ลดลง
- hardware จริงมี noise และ error ต่างกันในแต่ละ qubit/การเชื่อมต่อ
- ควรดู backend information และ error map เพื่อเลือก layout ที่ดี

## 12) Deutsch-Jozsa Algorithm
โจทย์:
- มี Boolean function ที่รับประกันว่าเป็น `constant` หรือ `balanced`
- ต้องตัดสินว่าเป็นแบบไหน

แบบคลาสสิก:
- อาจต้อง query หลายครั้ง

แบบควอนตัม:
- ใช้ oracle เพียงครั้งเดียวก็พอ

หลักการ:
- สร้าง superposition
- ให้ oracle ใส่ phase ตามค่า `f(x)`
- ใช้ interference
- ถ้าวัดแล้วได้ all-zero แปลว่า constant
- ถ้าได้อย่างอื่น แปลว่า balanced

ความสำคัญ:
- เป็นตัวอย่างแรก ๆ ของ quantum advantage ในแง่ query complexity

## 13) Grover's Algorithm
ใช้กับ `unstructured search`

แนวคิด:
- เริ่มจาก superposition ของทุกคำตอบ
- `oracle` กลับ phase ของคำตอบที่ต้องการ
- `amplitude amplification` หรือ inversion about the mean ช่วยเพิ่มโอกาสของคำตอบเป้าหมาย
- ทำซ้ำประมาณ `pi/4 * sqrt(N/t)` รอบ
  โดย `N` คือจำนวนสถานะ และ `t` คือจำนวนคำตอบเป้าหมาย

ผลลัพธ์:
- จากเดิมค้นหาเชิงเส้นประมาณ `N`
- ควอนตัมลดเหลือประมาณ `sqrt(N)`

## 14) Quantum Fourier Transform (QFT)
เป็น quantum version ของ discrete Fourier transform บน amplitudes

ใช้เปลี่ยนจาก computational basis ไปสู่ Fourier basis

QFT เป็นเครื่องมือกลางในหลาย algorithm โดยเฉพาะ QPE และ Shor

## 15) Quantum Phase Estimation (QPE)
เป้าหมาย:
- ประมาณค่า phase `theta` ที่อยู่ใน eigenvalue ของ unitary `U`

แนวคิดหลัก:
- ใช้ `phase kickback`
- เขียน phase ลงใน counting register
- ใช้ inverse QFT เพื่ออ่านค่า phase ออกมา

QPE เป็น primitive สำคัญในอัลกอริทึมควอนตัมระดับสูง

## 16) Shor's Algorithm
ใช้แยกตัวประกอบจำนวนเต็มขนาดใหญ่

ภาพรวม:
- ลดปัญหา factoring ให้กลายเป็น `order finding`
- ส่วนหนึ่งทำแบบคลาสสิก เช่น สุ่ม `a`, หา `gcd(a, N)`
- ส่วนควอนตัมใช้หาคาบของฟังก์ชัน `f(x) = a^x mod N`

ขั้นคลาสสิกโดยย่อ:
- สุ่ม `a < N`
- ถ้า `gcd(a, N) != 1` จบเลยเพราะเจอ factor
- ถ้าได้ period `r` แล้ว และ `r` เหมาะสม
- คำนวณ `gcd(a^(r/2) +- 1, N)` เพื่อหา factor

ส่วนควอนตัม:
- สร้างสถานะ superposition
- คำนวณฟังก์ชันเชิงควอนตัม
- ใช้ QFT เพื่อดึง periodicity
- วัดผลและวิเคราะห์ candidate ของ period

ความสำคัญ:
- เป็นเหตุผลใหญ่ที่ทำให้ quantum computing มีผลต่อ cryptography แบบ classical จำนวนมาก

## สรุปสั้น
- qubit ต่างจาก bit ตรงที่อยู่ใน superposition ได้
- การวัดใช้ Born rule และทำให้สถานะ collapse
- gate ควอนตัมเป็น unitary และ reversible
- entanglement เป็นทรัพยากรสำคัญ

## Q&A
- ถาม: superposition คืออะไร?
  ตอบ: คือสถานะที่ qubit อยู่ได้หลายความเป็นไปได้พร้อมกัน เช่น `alpha|0> + beta|1>` ยังไม่ยุบเป็น 0 หรือ 1 จนกว่าจะวัด
- ถาม: interference effect คืออะไร?
  ตอบ: คือการที่ amplitude ของสถานะควอนตัมเสริมกันหรือหักล้างกันตาม phase ทำให้บางคำตอบเด่นขึ้นและบางคำตอบหายไป ซึ่งเป็นกลไกสำคัญของ quantum algorithm
- teleportation และ superdense coding แสดงพลังของ entanglement
- algorithm สำคัญในบทนี้คือ Deutsch-Jozsa, Grover, QFT, QPE, Shor

## Q&A
ยังไม่มีคำถามในบทนี้
