# 1. Greedy Algorithm

## ภาพรวม
บทนี้อธิบายแนวคิด `greedy algorithm` ซึ่งเป็นกรอบการออกแบบ algorithm ที่เลือกสิ่งที่ “ดีที่สุดในก้าวนี้” โดยไม่ลองทุกทางเลือกและไม่มองลึกไปข้างหน้า จึงมักเร็ว แต่ใช้ได้ถูกต้องแค่บางปัญหาและต้องพิสูจน์เสมอ

## 1) แนวคิดของ Greedy
- สร้างคำตอบทีละขั้นเหมือน `brute force`
- แต่ไม่ enumerate ทุก candidate solution
- ในแต่ละขั้นจะเลือกทางเลือกที่ดีที่สุดเฉพาะตอนนั้น
- ความหมายของคำว่า “ดีที่สุด” ขึ้นกับปัญหา
- ข้อดีคือเร็ว เพราะไม่ลองทุกทางเลือก
- ข้อเสียคือไม่ได้การันตีว่าคำตอบจะถูกเสมอ

## 2) ทำไม Greedy อาจผิด
- การเลือกที่ดีที่สุดเฉพาะหน้า อาจทำให้คำตอบรวมไม่ดีที่สุด
- ตัวอย่างเช่นการเลือกถนนที่สั้นที่สุดในก้าวนี้ อาจพาไปเจอเส้นทางรวมที่ยาวกว่า
- ดังนั้น `greedy` ไม่ได้ถูกกับทุกปัญหา

สรุป:
- `local optimum` ไม่ได้แปลว่า `global optimum`
- ถ้าจะใช้ greedy ต้องมี proof ว่ากลยุทธ์นี้ถูกกับปัญหานั้นจริง

## 3) Rational Knapsack
เป็นรูปแบบหนึ่งของ `0-1 Knapsack` ที่ต่างกันตรง
- เลือก “บางส่วน” ของ item ได้
- ให้ `x_i` อยู่ในช่วง `0..1`
- เป้าหมายคือ maximize ค่ารวม โดยน้ำหนักรวมไม่เกิน `W`

แนวคิด greedy:
- คำนวณ `price / weight ratio`
- เรียง item จาก ratio มากไปน้อย
- หยิบ item ที่คุ้มที่สุดก่อน
- ถ้าหยิบทั้งชิ้นไม่ได้ ให้หยิบเป็น fraction เท่าที่เหลือ

ทำไมวิธีนี้ใช้ได้:
- เพราะเมื่อแบ่ง item ได้ คำตอบที่ดีที่สุดคือกินพื้นที่ด้วย item ที่คุ้มที่สุดก่อนเสมอ
- ต่างจาก `0-1 Knapsack` ที่บางครั้งต้องยอมเลือก item ที่ ratio ต่ำกว่าเพื่อให้พื้นที่ลงตัว

time complexity:
- sorting ใช้ `O(n log n)`
- การไล่เลือก item ใช้ `O(n)`

โค้ดตัวอย่าง (C++):
```cpp
#include <algorithm>
#include <vector>

struct Item {
    double weight;
    double price;
};

double fractionalKnapsack(double capacity, std::vector<Item> items) {
    std::sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        return (a.price / a.weight) > (b.price / b.weight);
    });

    double total = 0.0;
    for (const auto& item : items) {
        if (capacity == 0.0) break;

        double take = std::min(capacity, item.weight);
        total += take * (item.price / item.weight);
        capacity -= take;
    }
    return total;
}
```

## 4) Activity Selection
ปัญหาคือมีงานหลายงาน แต่ละงานมี `start` และ `stop` time
- ต้องเลือกงานให้ได้จำนวนมากที่สุด
- งานที่เลือกต้องไม่ทับเวลากัน

แนวคิด greedy:
- เรียงงานตาม `finish time` จากน้อยไปมาก
- เลือกงานที่จบเร็วที่สุดก่อน
- จากนั้นเลือกงานถัดไปที่เริ่มหลังจากงานล่าสุดจบ

เหตุผล:
- การเลือกงานที่จบเร็วที่สุด จะเหลือเวลาให้เลือกงานอื่นได้มากที่สุด

time complexity:
- sorting ใช้ `O(n log n)`
- การไล่เลือกงานใช้ `O(n)`
- รวมเป็น `O(n log n)`

โค้ดตัวอย่าง (C++):
```cpp
#include <algorithm>
#include <vector>

struct Activity {
    int start;
    int finish;
};

std::vector<Activity> selectActivities(std::vector<Activity> jobs) {
    std::sort(jobs.begin(), jobs.end(), [](const Activity& a, const Activity& b) {
        return a.finish < b.finish;
    });

    std::vector<Activity> selected;
    int lastFinish = -1;

    for (const auto& job : jobs) {
        if (job.start >= lastFinish) {
            selected.push_back(job);
            lastFinish = job.finish;
        }
    }
    return selected;
}
```

## 5) สิ่งที่ต้องจำเกี่ยวกับ Greedy
- เร็วเพราะไม่ลองทุกทางเลือก
- ใช้ได้เฉพาะบางปัญหา
- ต้องพิสูจน์ correctness เสมอ
- ตัวอย่างที่ใช้ได้ดีคือ `Fractional Knapsack` และ `Activity Selection`
- ตัวอย่างที่ใช้ไม่ได้คือ `0-1 Knapsack` ถ้าใช้ ratio แบบเดียวกัน
