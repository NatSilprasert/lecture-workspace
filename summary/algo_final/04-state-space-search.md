# 4. State Space Search

## ภาพรวม

บทนี้อธิบายแนวคิด `state space search` ซึ่งมองการหาคำตอบเป็นการเดินในต้นไม้ของ partial solutions แล้วใช้กลยุทธ์อย่าง `DFS`, `BFS`, `Backtracking`, และ `Branch and Bound` เพื่อสำรวจหรือ prune พื้นที่ค้นหา

## 1) Key Concept

- เวลาทำ brute force เราต้องกำหนด `search space`
- นอกจากกำหนดว่า candidate solution มีอะไรบ้างแล้ว ยังต้องคิดต่อว่า
  - จะ enumerate ในลำดับไหน
  - จะลดขนาด search space ได้ไหม

หัวข้อหลัก:

- `DFS`
- `BFS`
- `Backtracking`
- `Branch and Bound`

## 2) State Space Tree

`state space tree` คือ tree ที่แทนการสร้าง candidate solution ทีละขั้น

ชนิดของ state:

- `initial state` จุดเริ่มต้น
- `partial state` คำตอบบางส่วนที่ยังไม่สมบูรณ์
- `candidate solution` คำตอบเต็มที่ตรวจเงื่อนไขได้แล้ว

ประโยชน์:

- ใช้อธิบายลำดับการ search
- ใช้ตัดกิ่ง (`prune`) ที่ไม่มีทางไปสู่คำตอบที่ดีได้

## 3) 8-Queen Problem

โจทย์:

- วาง queen 8 ตัวบนกระดานหมากรุก
- queen ทุกคู่ต้องไม่กินกัน

ใจความสำคัญของบท:

- การออกแบบ representation มีผลมากต่อขนาด search space

เวอร์ชันของ search space:

### Version 0.1

- คิดแบบหยาบที่สุด
- queen 8 ตัว เลือกได้จาก 64 ช่อง
- search space ใหญ่มาก และยังมีการวางซ้อนกันได้

### Version 0.2

- ใช้การเลือก 8 ช่องจาก 64 ช่อง
- ตัดกรณีซ้อนกันออก
- ยังใหญ่มาก

### Version 0.3

- ใช้ observation ว่าแต่ละ row ควรมี queen แค่ตัวเดียว
- แทนคำตอบเป็น sequence ของ columns
- search space ลดเหลือ `8^8`

### Version 0.4

- ใช้ observation เพิ่มว่าแต่ละ column ก็ควรไม่ซ้ำ
- แทนคำตอบเป็น permutation ของ columns
- search space ลดเหลือ `8!`

บทเรียนสำคัญ:

- representation ที่ดีช่วยลด search space ได้มากแม้ยังไม่ใช้ pruning

ขนาด search space ที่ควรจำ:

- version 0.1 ประมาณ `64^8`
- version 0.2 ประมาณ `C(64, 8)`
- version 0.3 เท่ากับ `8^8`
- version 0.4 เท่ากับ `8!`

## 4) DFS และ BFS บน State Space Tree

### DFS

- ลงลึกก่อน
- ใช้ memory น้อยกว่า
- เหมาะกับ recursion และ backtracking

time complexity:

- เวลาเป็น `O(number of states)` ของต้นไม้ค้นหา
- memory มักเป็น `O(depth)`

### BFS

- ขยายทีละชั้น
- เหมาะเมื่ออยากได้คำตอบที่ตื้นที่สุดก่อน
- แต่ใช้ memory มากกว่า

time complexity:

- เวลาเป็น `O(number of states)`
- memory อาจสูงมากเพราะต้องเก็บ frontier ทั้งชั้น

โค้ดตัวอย่าง (C++): DFS สำหรับ 8-queen แบบ backtracking

```cpp
#include <iostream>
#include <vector>
#include <unordered_set>

using namespace std;

int n, result = 0;
int q[20], used[20];

bool check(int depth) {
    for (int i = 0; i < depth; i++) {
        if (abs(i - depth) == abs(q[i] - q[depth])) return false;
    }
    return true;
}

void solve(int row_idx) {
    if (row_idx == n) {
        result++;
        return;
    }
    for (int i = 0; i < n; i++) {
        if (used[i]) continue;
        q[row_idx] = i;
        used[i] = 1;
        if (check(row_idx)) solve(row_idx + 1);
        used[i] = 0;
    }
}

int main() {
    cin >> n;
    solve(0);
    cout << result;
    return 0;
}
```

## 5) Backtracking

- เป็นการ search พร้อม pruning
- ถ้า partial state ผิดเงื่อนไขแน่นอน ก็หยุดต่อทันที
- ใช้กับ constraint satisfaction problems ได้ดี

ตัวอย่าง:

- 8-queen
- subset sum

แนวคิด:

- “หยุดเมื่อรู้ว่าไปต่อแล้วไม่มีทางเป็นคำตอบ”

time complexity:

- worst case ยังอาจเป็น exponential
- แต่ปกติเร็วกว่าการลองทุก candidate เพราะมี pruning

## 6) Branch and Bound

- ใช้กับ `constraint optimization problem`
- คล้าย backtracking แต่เพิ่มแนวคิด `bound`
- ถ้า partial state นี้มีค่าดีสุดที่เป็นไปได้ ยังแพ้คำตอบที่มีอยู่แล้ว ก็ prune ได้

ต่างจาก backtracking:

- backtracking ตัดเพราะผิด constraint
- branch and bound ตัดเพราะไม่มีทางชนะคำตอบปัจจุบัน

เทคนิค `Bounding Heuristic` ที่ใช้บ่อย:

- `remaining sum bound`
  - ใช้กับโจทย์อย่าง `0-1 Knapsack`
  - เอาค่าของ item ที่ยังไม่ตัดสินใจมาบวกเป็นเพดานบนแบบหยาบ
  - ถ้า `sumP + remainingValue` ยังน้อยกว่าคำตอบที่ดีที่สุดตอนนี้ ก็ prune ได้
- `fractional knapsack bound`
  - ใช้กับ `0-1 Knapsack`
  - สมมติอย่างมองโลกดีว่า item ที่เหลือแบ่งหยิบแบบ fractional ได้
  - มักให้ upper bound ที่แน่นกว่า `remaining sum bound`
- `relaxation bound`
  - คลาย constraint ของปัญหาจริงให้เป็นปัญหาที่แก้ง่ายกว่า
  - คำตอบของปัญหาที่ถูกคลายจะใช้เป็น bound ของปัญหาจริง
  - ตัวอย่างคลาสสิกคือใช้ `fractional knapsack` เป็น relaxation ของ `0-1 knapsack`
- `known part + optimistic unknown part`
  - แยกคะแนนเป็นส่วนที่ทำมาแล้ว กับส่วนที่เหลือที่ “อาจดีที่สุด”
  - ถ้ารวมกันแล้วยังสู้คำตอบปัจจุบันไม่ได้ ก็หยุดได้
- `minimum additional cost bound`
  - ใช้กับโจทย์แบบ minimize
  - ประเมินว่าจาก state นี้อย่างน้อยต้องเสียเพิ่มอีกเท่าไร
  - ถ้า `currentCost + lowerBound >= bestAnswer` ก็ prune
- `maximum possible gain bound`
  - ใช้กับโจทย์แบบ maximize
  - ประเมินว่าจาก state นี้ต่อให้ดีที่สุดยังได้เพิ่มอีกเท่าไร
  - ถ้า `currentGain + upperBound <= bestAnswer` ก็ prune

หลักที่ต้องจำ:

- ถ้าโจทย์เป็น maximize และเราใช้ upper bound, heuristic ต้องไม่ต่ำกว่าความจริง
- ถ้าโจทย์เป็น minimize และเราใช้ lower bound, heuristic ต้องไม่สูงกว่าความจริง
- heuristic ควรคำนวณเร็วพอ ไม่อย่างนั้น cost ของ heuristic เองจะไม่คุ้ม

ตัวอย่างจากสไลด์:

- `Knapsack`
  - backtracking: หยุดเมื่อ `sumW > W`
  - branch and bound: หยุดเมื่อ `sumP + tail[step] < bestAnswer`
- `Coin Change`
  - ใช้แนวคิดประเมินจำนวนเหรียญที่เหลือแบบมองโลกดี เพื่อดูว่าต่อให้ดีที่สุดยังชนะคำตอบปัจจุบันได้ไหม

time complexity:

- worst case ยังอาจเป็น exponential
- ประสิทธิภาพจริงขึ้นกับ bound และลำดับการเลือก state

## 7) Least-Cost Search / Best-First Search

- ลำดับการเลือก partial state สำคัญมาก
- แทนที่จะใช้ stack หรือ queue อาจใช้ priority queue
- เลือก state ที่ดู “ดีที่สุด” จาก `Bounding Heuristic Function` ก่อน เช่น cost ต่ำสุด

ข้อดี:

- บางครั้งเจอคำตอบดีเร็ว

ข้อเสีย:

- มักใช้ memory มาก
- ถ้า heuristic ไม่ดี อาจไม่คุ้ม

time complexity:

- worst case ยังอาจเป็น exponential
- ถ้าใช้ priority queue จะมี cost เพิ่มจากการ push/pop แต่ละ state

## 8) สิ่งที่ต้องจำ

- `state space tree` แทนการสร้างคำตอบทีละขั้น
- representation ที่ดีช่วยลด search space ได้มาก
- `DFS` เหมาะกับ recursion และ backtracking
- `BFS` ขยายทีละชั้น แต่ใช้ memory มากกว่า
- `Backtracking` = prune เพราะผิด constraint
- `Branch and Bound` = prune เพราะไม่มีทางชนะคำตอบที่มีอยู่

## Q&A

- Q: มีตัวอย่างโจทย์แบบในสไลด์อะไรบ้าง?
A: หลัก ๆ คือ `8-Queen`, `Triple and Half`, `Sum of Subset`, และ `0-1 Knapsack` โดยแต่ละโจทย์ถูกใช้เพื่อสอนคนละมุมของ state space search เช่นการออกแบบ state, DFS/BFS, backtracking, และ branch and bound
- Q: เทคนิค `Bounding Heuristic` ที่ใช้บ่อยมีอะไรบ้าง?
A: ที่ใช้บ่อยคือ `remaining sum bound`, `fractional knapsack bound`, `relaxation bound`, และแนวคิด `known part + optimistic unknown part` โดยหลักคือ bound ต้องปลอดภัยพอที่จะใช้ prune ได้โดยไม่ตัดคำตอบที่ดีที่สุดทิ้ง

