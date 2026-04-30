# 3. Shortest Path in Graph

## ภาพรวม
บทนี้อธิบายปัญหา `shortest path` ใน graph โดยเน้น `single-source shortest path` และ `all-pairs shortest path` พร้อมอธิบาย algorithm หลักคือ `Dijkstra`, `Bellman-Ford`, และ `Floyd-Warshall`

## 1) ปัญหา Shortest Path
- ต้องการหา path จาก `u` ไป `v` ที่มีผลรวม weight น้อยที่สุด
- path ที่ดีที่สุดอาจไม่ใช่ path ที่มีจำนวนน้อย edge ที่สุด

### Single-Source Shortest Path
- ให้ graph `G` กับ starting node `s`
- ต้องหา `dist[x]` สำหรับทุก node `x`
- ถ้าต้องการกู้ path จริง มักใช้ `prev[x]`

## 2) ทำไม BFS ใช้ได้แค่บางกรณี
- `BFS` ให้ shortest path ได้ใน `unweighted graph`
- หรือกรณีที่ทุก edge มี weight เท่ากัน
- ถ้า graph มี weight ต่างกัน BFS ปกติจะไม่พอ

time complexity:
- `BFS` ใช้เวลา `O(n + e)`

## 3) Dijkstra’s Algorithm
แนวคิด:
- มองเหมือนแต่ละ node มี “alarm”
- node ที่มีระยะทางน้อยที่สุดจะถูก finalize ก่อน
- จากนั้นใช้ node นั้นไป `relax` เพื่อนบ้าน

ขั้นตอนหลัก:
1. กำหนด `dist[s] = 0`
2. node อื่นเริ่มที่ `INF`
3. ดึง node ที่ `dist` น้อยสุดจาก priority queue
4. ลองอัปเดต `dist[v]` ด้วย `dist[u] + w(u,v)`
5. ทำซ้ำจน queue หมด

ใช้ได้เมื่อ:
- edge weights ต้องไม่เป็นลบ
- ใช้ได้ทั้ง directed และ undirected graph ถ้าไม่มี negative edge

time complexity:
- ถ้าใช้ heap มักอยู่ที่ `O((n + e) log n)`

โค้ดตัวอย่าง (C++):
```cpp
#include <queue>
#include <utility>
#include <vector>

using Edge = std::pair<int, int>; // {to, weight}

std::vector<int> dijkstra(int start, const std::vector<std::vector<Edge>>& adj) {
    const int INF = 1e9;
    std::vector<int> dist(adj.size(), INF);
    std::priority_queue<
        std::pair<int, int>,
        std::vector<std::pair<int, int>>,
        std::greater<>
    > pq;

    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        auto [du, u] = pq.top();
        pq.pop();
        if (du != dist[u]) continue;

        for (auto [v, w] : adj[u]) {
            if (dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

## 4) ปัญหา Negative Edge
- ถ้า graph มี edge weight ติดลบ `Dijkstra` อาจตอบผิด
- เพราะสมมติฐานสำคัญของ Dijkstra คือ node ที่ถูก finalize แล้วจะไม่มีทางดีขึ้นอีก
- สมมติฐานนี้พังเมื่อมี negative edge

เพิ่มเติม:
- undirected graph ที่มี negative edge จะทำให้ shortest path ไม่มีความหมาย เพราะอาจวนลดค่าได้เรื่อย ๆ
- directed graph ยังพอมี negative edge ได้ ถ้าไม่มี `negative cycle`

## 5) Bellman-Ford Algorithm
แนวคิด:
- shortest path ที่ไม่มี cycle ใช้ edge ได้ไม่เกิน `n - 1` เส้น
- จึงผ่อนคลาย (`relax`) edges ทั้งหมดซ้ำ `n - 1` รอบ

ข้อดี:
- ใช้กับ directed graph ที่มี negative edge ได้
- ตรวจ `negative cycle` ได้

time complexity:
- เวลา `O(n e)`
- memory `O(n)`

ขั้นตอน:
1. กำหนด `dist[s] = 0`
2. ทำซ้ำ `n - 1` รอบ
3. ทุกรอบ ไล่ทุก edge `(u, v, w)` แล้ว relax
4. ถ้ามีการ relax ได้อีกในรอบที่ `n` แปลว่ามี negative cycle

โค้ดตัวอย่าง (C++):
```cpp
#include <tuple>
#include <vector>

struct BFResult {
    std::vector<int> dist;
    bool hasNegativeCycle;
};

BFResult bellmanFord(int n, int start, const std::vector<std::tuple<int, int, int>>& edges) {
    const int INF = 1e9;
    std::vector<int> dist(n, INF);
    dist[start] = 0;

    for (int i = 0; i < n - 1; ++i) {
        for (auto [u, v, w] : edges) {
            if (dist[u] != INF && dist[v] > dist[u] + w) {
                dist[v] = dist[u] + w;
            }
        }
    }

    for (auto [u, v, w] : edges) {
        if (dist[u] != INF && dist[v] > dist[u] + w) {
            return {dist, true};
        }
    }
    return {dist, false};
}
```

## 6) Floyd-Warshall
- ใช้หา shortest path ระหว่างทุกคู่ node (`all-pairs shortest path`)
- เป็นแนวคิดแบบ dynamic programming
- ให้ `dist[i][j]` คือระยะจาก `i` ไป `j`
- ลองอนุญาตให้ node `k` เป็นตัวกลางเพิ่มเข้ามาทีละตัว

recurrence หลัก:
- `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`

ข้อดี:
- โค้ดสั้นและตรงไปตรงมา
- ใช้กับ negative edge ได้
- ตรวจ negative cycle ได้จาก `dist[i][i] < 0`

ข้อเสีย:
- ใช้เวลา `O(n^3)`
- ใช้ memory `O(n^2)`

โค้ดตัวอย่าง (C++):
```cpp
#include <vector>

void floydWarshall(std::vector<std::vector<int>>& dist, int INF) {
    int n = dist.size();
    for (int k = 0; k < n; ++k) {
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (dist[i][k] == INF || dist[k][j] == INF) continue;
                if (dist[i][j] > dist[i][k] + dist[k][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
}
```

## 7) เปรียบเทียบ Algorithm

### Dijkstra
- เร็ว
- ใช้กับ non-negative edge
- เหมาะกับ single-source shortest path
- เวลา `O((n + e) log n)` เมื่อใช้ heap

### Bellman-Ford
- ช้ากว่า
- รองรับ negative edge
- ตรวจ negative cycle ได้
- เวลา `O(n e)`

### Floyd-Warshall
- ใช้กับ all-pairs shortest path
- ง่ายและตรงไปตรงมา
- time `O(n^3)`
- memory `O(n^2)`

## 8) สิ่งที่ต้องจำ
- `BFS` ใช้กับ shortest path ได้เมื่อ graph ไม่มี weight
- `Dijkstra` ห้ามมี negative edge
- `Bellman-Ford` รองรับ negative edge และเช็ก negative cycle ได้
- `Floyd-Warshall` ใช้กับ shortest path ทุกคู่ node
