# 4. Storage Architecture

## ภาพรวม
บทนี้สรุปสถาปัตยกรรมของระบบเก็บข้อมูล ตั้งแต่เทคโนโลยีตัวสื่อเก็บข้อมูล, การทำ storage virtualization, แนวคิด RAID, และ storage ผ่าน network แบบ `SAN` กับ `NAS` รวมถึงประเด็นใช้งานจริง เช่น performance, redundancy และ sharing

## 1) Storage Technology
สื่อเก็บข้อมูลหลักในบทนี้มี 2 กลุ่ม

- `SSD` ใช้ flash memory
  เร็วกว่า ไม่มีชิ้นส่วนกลไก
- `HDD` ใช้จานหมุนเชิงกล
  ความจุสูง ราคาต่อ GB มักถูกกว่า แต่ช้ากว่า

## 2) Interface/Protocol Technology
สไลด์แยกให้เห็นว่าตัว "สื่อเก็บข้อมูล" กับ "interface/protocol" ไม่เหมือนกัน

ตัวอย่าง interface:
- `SAS` มักใช้ใน server
- `SATA` ใช้แพร่หลายกับ storage ทั่วไป
- `PCIe` ให้ bandwidth สูงมาก เหมาะกับ SSD สมัยใหม่
- `Fibre Channel` ใช้ในระบบองค์กร/ศูนย์ข้อมูล
- `USB`
- `Parallel ATA`
- `SCSI`

ประเด็นสำคัญ:
- storage technology บอกว่าเก็บข้อมูลอย่างไร
- interface บอกว่าสื่อสารกับอุปกรณ์อย่างไร

## 3) Storage Virtualization
คือการทำ abstraction ให้ระบบมอง physical storage หลายก้อนเป็น logical storage ก้อนเดียวหรือหลายก้อนตามต้องการ

ประโยชน์:
- จัดการง่ายขึ้น
- รวมหลาย disk เป็น pool เดียว
- ยืดหยุ่นต่อการขยายระบบ

ตัวอย่าง:
- physical disks หลายลูก แต่ระบบปฏิบัติการมองเป็น volume เดียว

## 4) RAID คืออะไร
`RAID = Redundant Array of Independent Disks`
 คือการรวมดิสก์หลายลูกให้ทำงานเป็นระบบเดียว เพื่อเพิ่มความเร็ว เพิ่มความทนทาน หรือเพิ่มความจุ โดยแต่ละ RAID level จะแลกกันระหว่าง performance, redundancy และพื้นที่ใช้ได้จริง

แนวคิด:
- รวม disk หลายลูกเพื่อแลกเปลี่ยนระหว่าง
  - performance
  - reliability
  - availability
  - capacity

ประวัติ:
- ถูกเสนอโดย David Patterson, Garth Gibson, Randy Katz ในปี 1987
- เดิมใช้แนวคิดว่าดิสกราคาทั่วไปหลายลูก รวมกันแล้วให้ทั้งความจุและความเร็วใกล้ระบบแพง ๆ ได้

## 5) RAID 0
`Striping`

ลักษณะ:
- กระจายข้อมูลข้ามหลาย disk
- ไม่มี redundancy

ข้อดี:
- performance สูง

ข้อเสีย:
- ถ้า disk ลูกเดียวเสีย ข้อมูลทั้งชุดอาจเสีย

จำนวน disk ขั้นต่ำ:
- 2 ลูก

ทำไมไม่ใช่ 1 ลูก:
- เพราะ RAID 0 คือการ `stripe` (กระจาย block) ข้ามหลายดิสก์เพื่ออ่าน/เขียนขนาน
- ถ้ามีแค่ 1 ลูก จะไม่มีอะไรให้กระจาย เท่ากับเป็นดิสก์เดี่ยวธรรมดา ไม่ใช่ RAID 0 จริง ๆ

ตัวอย่างใช้งาน:
- งาน scratch space ที่เน้นเร็วมากและยอมรับความเสี่ยงได้

## 6) RAID 1
`Mirroring`

ลักษณะ:
- เขียนข้อมูลซ้ำเป็นสำเนา

ข้อดี:
- อ่านได้ดี
- ทนต่อการเสียของ disk ได้ 1 ลูกในคู่ mirror
- reliability สูง

ข้อเสีย:
- ใช้พื้นที่คุ้มไม่ดี เพราะต้องเก็บซ้ำ

จำนวน disk ขั้นต่ำ:
- 2 ลูก

## 7) RAID 2
ลักษณะ:
- ใช้ bit-level parity
- ใช้ Hamming code

ข้อสังเกต:
- แทบไม่ใช้จริงในปัจจุบัน

อธิบายเพิ่ม:
- `bit-level parity` คือคำนวณ parity รายบิต (ตำแหน่งบิตเดียวกันจากหลายดิสก์)
- RAID 2 ใช้แนวคิด `Hamming code` เพื่อทำ error detection/correction ได้ละเอียดกว่า parity ธรรมดา
- แต่ต้องซิงก์ดิสก์หลายลูกและฮาร์ดแวร์ซับซ้อน จึงไม่ค่อยนิยมแล้ว

## 8) RAID 3
ลักษณะ:
- ใช้ byte-level parity
- เหมาะกับการอ่าน/เขียนยาว ๆ ต่อเนื่อง

ข้อเสีย:
- ไม่ดีสำหรับ random read/write
- ใช้งานจริงไม่แพร่หลาย

อธิบายเพิ่ม:
- `byte-level parity` คือคำนวณ parity เป็นหน่วย byte (หรือ small chunk) ข้ามดิสก์ข้อมูล
- มี parity disk แยกเหมือน RAID 4 แต่ละเอียดกว่าระดับ block

## 9) RAID 4
ลักษณะ:
- block-level striping
- parity แยกไว้ที่ disk เฉพาะลูก

ข้อดี:
- อ่านดี

ข้อเสีย:
- เขียนช้า เพราะ parity disk เป็นคอขวด

## 10) RAID 5
ลักษณะ:
- block-level parity แบบกระจาย
- parity ไม่ไปรวมอยู่ลูกเดียว

ข้อดี:
- balance ระหว่าง performance กับ redundancy
- ทน disk เสียได้ 1 ลูก
- เขียนดีกว่า RAID 4 เพราะไม่มี parity bottleneck ลูกเดียวชัด ๆ

จำนวน disk ขั้นต่ำ:
- 3 ลูก

Parity ทำงานอย่างไร (พื้นฐาน):
- นิยมใช้ `XOR parity`
- สมมติข้อมูล 3 ลูกเป็น `D1, D2, D3` จะได้ `P = D1 XOR D2 XOR D3`
- ถ้าดิสก์ข้อมูลเสีย 1 ลูก เช่น D2 หาย กู้คืนได้จาก `D2 = P XOR D1 XOR D3`
- นี่คือเหตุผลที่ RAID 4/5/6 ทนดิสก์เสียได้ตามจำนวน parity ที่มี

ตัวอย่างสั้น (บิตเดียว):
- D1=1, D2=0, D3=1 => P = 1 XOR 0 XOR 1 = 0
- ถ้า D2 หาย: D2 = P XOR D1 XOR D3 = 0 XOR 1 XOR 1 = 0 (กู้คืนได้)

สรุปความต่าง bit/byte/block parity:
- `bit-level` (RAID 2): ทำละเอียดระดับบิต มักพ่วง Hamming code
- `byte-level` (RAID 3): parity ระดับไบต์
- `block-level` (RAID 4/5): parity ระดับบล็อก เหมาะระบบสมัยใหม่กว่า

## 11) RAID 6
ลักษณะ:
- มี parity 2 ชุด

ข้อดี:
- ทน disk เสียพร้อมกันได้ 2 ลูก

ข้อเสีย:
- มี overhead มากกว่า RAID 5

## 12) Nested RAID
คือการผสมหลาย RAID levels เช่น:
- `RAID 01`
- `RAID 10`
- `RAID 50`

ใช้เพื่อหาจุดสมดุลเฉพาะงาน เช่น อยากได้ทั้งเร็วและทนทานขึ้น

จำนวนดิสก์ขั้นต่ำที่ใช้บ่อย:
- `RAID 0` = 2
- `RAID 1` = 2
- `RAID 2` = 3+ (เชิงทฤษฎี/ไม่ค่อยใช้จริง)
- `RAID 3` = 3
- `RAID 4` = 3
- `RAID 5` = 3
- `RAID 6` = 4
- `RAID 10` = 4
- `RAID 01` = 4
- `RAID 50` = 6 (อย่างน้อย 2 กลุ่ม RAID 5)

## 13) JBOD
`Just a Bunch Of Disks`

ลักษณะ:
- เอาหลาย disk มารวมกันแบบง่าย ๆ
- อาจใช้การ span
- รวม disk ต่างขนาดได้

ข้อควรระวัง:
- ไม่ได้หมายความว่ามี redundancy

## 14) Network Storage
สไลด์เน้นความต่างระหว่าง `block` กับ `file`

`Block storage`
- client เห็นเป็น block/sector
- client จัดการ filesystem เอง

`File storage`
- server จัดการ filesystem
- client เข้าถึงเป็นไฟล์/โฟลเดอร์

สรุปให้เห็นภาพ:
- `Block storage` เหมือนได้ "ฮาร์ดดิสก์ดิบ" มา 1 ก้อน แล้วเครื่องปลายทางเป็นคนฟอร์แมต/สร้าง filesystem เอง
- `File storage` เหมือนได้ "โฟลเดอร์แชร์" ที่ฝั่ง storage จัดการ filesystem ให้แล้ว ผู้ใช้เปิดเป็นไฟล์ได้ทันที

นอกจากนี้ยังมี:
- `Object storage`
  สไลด์พูดถึงว่ามีอยู่จริง แม้ไม่ได้ลงรายละเอียด

ตัวอย่าง cloud:
- Amazon EBS = block
- Amazon S3 = object
- Google Persistent Disk = block
- Google Cloud Filestore = file

ตัวอย่างงานจริง:
- `Block storage` เหมาะกับฐานข้อมูล, VM boot disk, งานที่ต้องการ IOPS/latency คุมละเอียด
- `File storage` เหมาะกับแชร์ไฟล์ในทีม, home directory, งานเอกสาร/สื่อที่หลายคนเข้าถึงร่วมกัน

## 15) SAN
`Storage Area Network`

ลักษณะ:
- เป็น block storage ผ่าน network
- มักใช้ iSCSI หรือเทคโนโลยีใกล้เคียง
- มักต้องใช้อุปกรณ์เฉพาะและระบบเชื่อมต่อระดับองค์กร

สิ่งที่ client มองเห็น:
- เห็นเป็น block device
- ฝั่ง client/OS ต้องจัดการ filesystem เอง

ข้อสังเกต:
- มักอุทิศให้เครื่องใดเครื่องหนึ่งมากกว่าแชร์พร้อมกันแบบง่าย ๆ

## 16) NAS
`Network Attached Storage`

ลักษณะ:
- เป็น file storage
- เหมือน network drive

ข้อดี:
- แชร์ได้ง่ายระหว่างหลาย client
- server จัดการ filesystem และ permission ให้

protocol ที่พบบ่อย:
- `NFS`
- `CIFS/SMB`
- `AFP` แบบเก่า
- `FTP`, `SFTP`

## 17) SAN vs NAS แบบสั้น
`SAN`
- ให้ block
- client ดูแล filesystem เอง
- มักใช้ในงานที่อยากให้ระบบปลายทางควบคุม storage โดยตรง

`NAS`
- ให้ file
- server ดูแล filesystem
- เหมาะกับการแชร์ไฟล์หลายผู้ใช้

## 18) Deduplication
แนวคิดคือถ้ามี block หรือไฟล์ซ้ำกันมาก อาจเก็บเพียงชุดเดียวแล้วอ้างร่วมกัน

ประโยชน์:
- ประหยัดพื้นที่จริง

เหมาะกับ:
- backup
- VM images
- ระบบองค์กรที่มีไฟล์ซ้ำเยอะ

## 19) Real-World Considerations
เวลาจะเลือกสถาปัตยกรรม storage ต้องดูหลายปัจจัยพร้อมกัน

- performance
- price
- block vs file vs object
- sharing
- reliability
- redundancy
- deduplication

ตัวอย่าง:
- ระบบฐานข้อมูลมักชอบ block storage ที่ควบคุมได้ละเอียด
- ระบบแชร์เอกสารในทีมมักเหมาะกับ NAS หรือ object/file storage

## 20) ประเด็นเชิงวิเคราะห์จากสไลด์
- RAID 5 เร็วกว่า RAID 4 เพราะ parity กระจาย ไม่เกิดคอขวดที่ parity disk ลูกเดียว
- user experience ของ cloud drive บางเจ้าดูเหมือน "ไฟล์อยู่ในเครื่อง" ทั้งที่จริงอาจใช้ sync, cache, on-demand fetch และ abstraction หลายชั้น

## สรุปสั้น
- storage architecture ไม่ได้มีแค่เลือก SSD หรือ HDD แต่รวมถึง protocol, virtualization, redundancy และรูปแบบการเข้าถึง
- RAID ใช้หลาย disk เพื่อแลก performance, capacity และความทนทาน
- SAN คือ block storage ผ่าน network
- NAS คือ file storage ผ่าน network
- งานจริงต้องเลือกตามรูปแบบ workload และข้อจำกัดด้านต้นทุน/ความน่าเชื่อถือ

## Q&A
- ถาม: RAID คืออะไร?
  ตอบ: `RAID` คือการรวมดิสก์หลายลูกให้ทำงานเป็นระบบเดียว เพื่อเพิ่มความเร็ว เพิ่มความทนทาน หรือเพิ่มความจุ โดยแต่ละ RAID level จะแลกกันระหว่าง performance, redundancy และพื้นที่ใช้ได้จริง
- ถาม: ทำไม RAID 5 เร็วกว่า RAID 4?
  ตอบ: เพราะ RAID 4 เก็บ parity ไว้ดิสก์ลูกเดียวทำให้เกิดคอขวดตอนเขียน แต่ RAID 5 กระจาย parity ทุกดิสก์ จึงกระจายภาระเขียนและเขียนได้ดีกว่า
- ถาม: ทำไม RAID 0 ต้องมีขั้นต่ำ 2 ลูก ไม่ใช่ 1 ลูก?
  ตอบ: เพราะ RAID 0 ต้องทำ striping ข้ามหลายดิสก์เพื่อความเร็ว ถ้ามีแค่ 1 ลูกจะไม่มีการกระจายข้อมูล จึงเป็นแค่ดิสก์เดี่ยว ไม่ถือว่าเป็น RAID 0
- ถาม: bit/byte/block parity ทำงานยังไง และเกี่ยวกับ Hamming code ยังไง?
  ตอบ: parity คือข้อมูลตรวจสอบที่คำนวณจากข้อมูลจริง (มักใช้ XOR) เพื่อกู้ข้อมูลเมื่อดิสก์เสียบางส่วน; bit-level parity ละเอียดสุดและ RAID 2 ใช้ Hamming code เพื่อช่วยตรวจ/แก้บิตผิดพลาด, ส่วน byte/block parity (RAID 3/4/5) เน้นความง่ายและใช้งานจริงมากกว่า
- ถาม: แต่ละ RAID ใช้ดิสก์ขั้นต่ำเท่าไหร่?
  ตอบ: โดยทั่วไป RAID 0=2, RAID 1=2, RAID 2=3+, RAID 3=3, RAID 4=3, RAID 5=3, RAID 6=4, RAID 10=4, RAID 01=4, RAID 50=6 (ขึ้นกับจำนวนกลุ่ม RAID 5)
- ถาม: block storage vs file storage ต่างกันยังไง พร้อมตัวอย่าง?
  ตอบ: block คือให้ก้อนดิสก์ดิบให้เครื่องปลายทางจัดการ filesystem เอง (เช่น Amazon EBS, Google Persistent Disk) เหมาะกับ DB/VM; file คือให้แชร์เป็นไฟล์/โฟลเดอร์โดย server ดูแล filesystem (เช่น NAS, Google Filestore) เหมาะกับงานแชร์ไฟล์ทีม
