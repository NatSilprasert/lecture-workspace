# 5. NP-Complete

## ภาพรวม
บทนี้อธิบายแนวคิดเรื่อง “ความยากของปัญหา” ในมุมมองของ computer science โดยเน้น complexity classes เช่น `P`, `NP`, `NP-Hard`, `NP-Complete`, และ `Undecidable` รวมถึงแนวคิด `reduction`

## 1) Easy vs Hard Problem
เวลาเราถามว่าปัญหายากไหม เราหมายถึง
- ยากสำหรับมนุษย์ออกแบบ algorithm
- หรือยากสำหรับคอมพิวเตอร์ในการรัน algorithm

บทนี้สนใจความหมายแบบหลัง:
- ใช้เวลาและทรัพยากรมากแค่ไหน

intuition แบบง่าย:
- ปัญหา “ง่าย” มักแก้ได้ในเวลา polynomial
- ปัญหา “ยาก” คือปัญหาที่เราไม่รู้ polynomial-time algorithm

## 2) Unsolvable / Undecidable
- บางปัญหาไม่ใช่แค่ยาก แต่ “แก้ไม่ได้เลย”
- เรียกว่า `undecidable problem`
- หมายถึงไม่มี algorithm ที่ถูกต้องสำหรับทุก input

ตัวอย่างสำคัญ:
- `Halting Problem`

โจทย์:
- รับ program `P` และ input `S`
- ถามว่า `P(S)` จะหยุดหรือวนลูปตลอดไป

ใจความของ proof:
- สมมติว่ามี algorithm ที่ตอบได้
- สร้าง program ที่ใช้ algorithm นั้นย้อนแย้งกับตัวเอง
- จึงสรุปว่า halting problem undecidable

## 3) Decision Problem vs Function Problem

### Decision Problem
- output มีแค่ `YES/NO`
- ใช้เป็นฐานในการจัด complexity classes

ตัวอย่าง:
- graph นี้มี Euler circuit ไหม
- จำนวนนี้เป็น prime ไหม

### Function Problem
- output ไม่ได้มีแค่ yes/no
- เช่นหา shortest path, หา largest element

ใจความสำคัญ:
- function problem มักแปลงไปเป็น decision problems ได้
- จึงมักวิเคราะห์ complexity ผ่าน decision version

## 4) Reduction
`Reduction` คือการแก้ปัญหา A โดยแปลงให้เป็นปัญหา B

ถ้า:
- แปลง input ของ A ไปเป็น input ของ B ได้ในเวลา polynomial
- และแปลง output ของ B กลับมาตอบ A ได้ในเวลา polynomial

เราจะบอกว่า
- `A` polynomially reducible to `B`

ความหมายเชิง hardness:
- ถ้า `A` reduce ไป `B` ได้
- แปลว่า `B` ไม่ง่ายกว่า `A`
- ถ้าแก้ `B` ได้ดี ก็ใช้แก้ `A` ได้ด้วย

ตัวอย่าง:
- `kth smallest` reduce ไป `sorting`
- sort ก่อน แล้วตอบ `S[k]`

time complexity ของ reduction ตัวอย่าง:
- sorting ใช้ `O(n log n)`
- อ่านคำตอบตำแหน่งที่ `k` ใช้ `O(1)`

โค้ดตัวอย่าง (C++):
```cpp
#include <algorithm>
#include <vector>

int kthSmallest(std::vector<int> a, int k) {
    std::sort(a.begin(), a.end());
    return a[k];
}
```

## 5) Complexity Classes หลัก

### P
- set ของปัญหาที่มี polynomial-time algorithm
- มักมองว่าเป็นปัญหา “ง่าย”

ตัวอย่าง:
- sorting
- shortest path
- MST

### NP
นิยามที่สำคัญ:
- ถ้าคำตอบเป็น `YES` และมี certificate ที่ถูกต้อง
- เราสามารถ verify ได้ในเวลา polynomial

intuition:
- ปัญหาที่ “ตรวจคำตอบ” ได้เร็ว แม้อาจ “หาคำตอบ” ไม่เร็ว

ตัวอย่างใน NP:
- `TSP` แบบ decision
- `0-1 Knapsack` แบบ decision
- graph coloring

ความสัมพันธ์:
- `P ⊆ NP`
- ทุกปัญหาใน P ย่อม verify ได้เร็วด้วย

## 6) NP-Hard
- ปัญหาที่อย่างน้อยยากเท่ากับทุกปัญหาใน NP
- ถ้าปัญหา NP ทุกตัว reduce มาที่มันได้ ปัญหานั้นเป็น `NP-Hard`
- ไม่จำเป็นต้องเป็น decision problem
- ไม่จำเป็นต้องอยู่ใน NP

## 7) NP-Complete
- คือปัญหาที่อยู่ใน `NP`
- และเป็น `NP-Hard` ด้วย

แปลว่า:
- verify ได้เร็ว
- และเป็นกลุ่มปัญหาที่ยากที่สุดใน NP

ความสำคัญ:
- ถ้าเราเจอ polynomial-time algorithm สำหรับปัญหา NP-complete แค่ตัวเดียว
- จะตามมาว่า `P = NP`

ปัจจุบัน:
- เรายังไม่รู้ว่า `P = NP` หรือไม่
- แต่โดยทั่วไปเชื่อว่า `P != NP`

## 8) วิธีพิสูจน์ว่าโจทย์เป็น NP-Complete
ขั้นตอนมาตรฐาน:
1. แสดงว่าปัญหาอยู่ใน `NP`
2. เลือกปัญหาที่รู้แล้วว่า `NP-Complete`
3. reduce ปัญหานั้นมายังปัญหาใหม่ในเวลา polynomial

ถ้าทำได้ครบ:
- ปัญหาใหม่เป็น `NP-Complete`

## 9) สิ่งที่ต้องจำ
- `Undecidable` = ไม่มี algorithm ที่ถูกต้องสำหรับทุก input
- `P` = แก้ได้เร็ว
- `NP` = ตรวจคำตอบ yes ได้เร็ว
- `NP-Hard` = อย่างน้อยยากเท่าทุกปัญหาใน NP
- `NP-Complete` = อยู่ใน NP และ NP-Hard
- เครื่องมือสำคัญที่สุดในการเปรียบเทียบความยากคือ `reduction`
