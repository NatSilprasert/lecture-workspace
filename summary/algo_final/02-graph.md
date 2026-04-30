# 2. Graph Algorithm

## ภาพรวม

บทนี้ปูพื้นฐาน graph และ algorithm สำคัญที่เกี่ยวข้อง เช่น `DFS`, `BFS`, `connected components`, `cycle detection`, `topological sort`, และ `minimum spanning tree (MST)`

## 1) Graph Recap

- graph เขียนเป็น `G = (V, E)`
- `V` คือ set ของ vertices
- `E` คือ set ของ edges
- edge แทนความเชื่อมต่อระหว่าง vertices

คำสำคัญ:

- `undirected graph` ขอบไม่มีทิศ
- `directed graph` ขอบมีทิศ
- `weighted graph` ขอบมีน้ำหนัก
- `path` คือลำดับของ node ที่เดินต่อกันได้
- `simple path` คือ path ที่ไม่มี node ซ้ำ
- `circuit` คือ path ที่เริ่มและจบ node เดียวกัน
- `degree` คือจำนวน edges ที่ต่อกับ node
- `simple graph` ไม่มี self-loop และไม่มี parallel edges

## 2) โครงสร้างข้อมูลสำหรับ Graph

operation หลัก:

- `nodes()` ดู node ทั้งหมด
- `adj(v)` ดูเพื่อนบ้านของ `v`
- `has_edge(u, v)` ตรวจว่ามี edge หรือไม่

โครงสร้างหลัก:

### Adjacency Matrix

- ใช้ array `A[n][n]`
- `A[x][y] = 1` หรือเก็บ weight ถ้ามี edge
- ตรวจ `has_edge` ได้เร็ว `O(1)`
- แต่กิน memory มาก `O(n^2)`

### Adjacency List

- เก็บ list ของเพื่อนบ้านสำหรับแต่ละ node
- เหมาะกับ graph ที่ edge ไม่หนาแน่น
- ประหยัด memory กว่า
- การไล่เพื่อนบ้านทำได้ดี

สรุป:

- graph หนาแน่น มักเหมาะกับ `adjacency matrix`
- graph เบาบาง มักเหมาะกับ `adjacency list`

## 3) DFS และ BFS

### DFS (Depth First Search)

- ไปให้ลึกที่สุดก่อนแล้วค่อยย้อนกลับ
- ใช้ `stack` หรือ recursion
- ใช้ตรวจ connectivity, cycle, topological sort ได้ดี

time complexity:
- เวลา `O(n + e)`
- memory `O(n)`

โค้ดตัวอย่าง (C++):

```cpp
#include <vector>

void dfs(int u, const std::vector<std::vector<int>>& adj, std::vector<bool>& visited) {
    visited[u] = true;
    for (int v : adj[u]) {
        if (!visited[v]) dfs(v, adj, visited);
    }
}
```

### BFS (Breadth First Search)

- ขยายทีละชั้นตามระยะจากต้นทาง
- ใช้ `queue`
- เหมาะกับการหาระยะสั้นสุดใน `unweighted graph`

time complexity:
- เวลา `O(n + e)`
- memory `O(n)`

โค้ดตัวอย่าง (C++):

```cpp
#include <queue>
#include <vector>

std::vector<int> bfsDistance(int start, const std::vector<std::vector<int>>& adj) {
    std::vector<int> dist(adj.size(), -1);
    std::queue<int> q;

    dist[start] = 0;
    q.push(start);

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v : adj[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
    return dist;
}
```

## 4) Connected Components

- ใน `undirected graph` คือกลุ่ม node ที่เดินถึงกันได้
- หาได้โดยรัน `DFS` หรือ `BFS` จากทุก node ที่ยังไม่ถูก visit
- จำนวนครั้งที่ต้องเริ่ม traversal ใหม่ = จำนวน connected components

time complexity:
- รวมทั้งกระบวนการเป็น `O(n + e)` เมื่อใช้ adjacency list

## 5) Cycle Detection และ Topological Sort

### Cycle Detection

- ใน undirected graph ใช้ DFS แล้วดูว่าเจอเพื่อนบ้านที่ถูก visit แล้วแต่ไม่ใช่ parent หรือไม่
- ใน directed graph มักใช้สถานะ 3 สีหรือ recursion stack

### Topological Sort

- ใช้กับ `Directed Acyclic Graph (DAG)` เท่านั้น
- ให้ลำดับ node ที่ทุก edge ชี้จากตัวก่อนหน้าไปตัวหลัง
- ถ้ามี cycle จะทำ topological sort ไม่ได้

time complexity:
- แบบ DFS ใช้เวลา `O(n + e)`

โค้ดตัวอย่าง (C++): DFS topological sort

```cpp
#include <algorithm>
#include <vector>

void topoDfs(int u, const std::vector<std::vector<int>>& adj,
             std::vector<bool>& visited, std::vector<int>& order) {
    visited[u] = true;
    for (int v : adj[u]) {
        if (!visited[v]) topoDfs(v, adj, visited, order);
    }
    order.push_back(u);
}

std::vector<int> topologicalSort(const std::vector<std::vector<int>>& adj) {
    std::vector<bool> visited(adj.size(), false);
    std::vector<int> order;

    for (int i = 0; i < (int)adj.size(); ++i) {
        if (!visited[i]) topoDfs(i, adj, visited, order);
    }

    std::reverse(order.begin(), order.end());
    return order;
}
```

## 6) Minimum Spanning Tree (MST)

- ใช้กับ weighted undirected graph
- เป้าหมายคือเลือก edges ให้เชื่อมทุก node
- ต้อง connected
- ไม่มี cycle
- ใช้ edge ทั้งหมด `n - 1` เส้น
- น้ำหนักรวมต่ำที่สุด

### Kruskal’s Algorithm

- เรียง edges ตามน้ำหนักจากน้อยไปมาก
- เลือก edge ถัดไปถ้าไม่ทำให้เกิด cycle
- ใช้ `Disjoint Set Union (DSU)` ช่วยเช็ก component

time complexity:
- sorting edges ใช้ `O(e log e)`
- DSU ทำให้ขั้นตอนเชื่อมและเช็กเร็วมาก
- โดยรวมมักเขียนเป็น `O(e log e)` หรือ `O(e log n)`

โค้ดตัวอย่าง (C++):

```cpp
#include <algorithm>
#include <numeric>
#include <vector>

struct Edge {
    int u, v, w;
};

struct DSU {
    vector<int> parent;

    DSU(int n) {
        parent.resize(n);
        for (int i = 0; i < n; i++) {
            parent[i] = i; // ตอนแรกทุกคนเป็นหัวหน้าตัวเอง
        }
    }

    int find(int x) {
        if (parent[x] == x) return x;
        return find(parent[x]);
    }

    bool unite(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);

        // ถ้าหัวหน้าเดียวกัน แปลว่าอยู่กลุ่มเดียวกันแล้ว
        // ถ้าเชื่อมอีกจะเกิด cycle
        if (rootA == rootB) return false;

        // รวมกลุ่ม b เข้า a
        parent[rootB] = rootA;

        return true;
    }
};

int kruskal(int n, std::vector<Edge> edges) {
    std::sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
        return a.w < b.w;
    });

    DSU dsu(n);
    int total = 0;
    for (const auto& e : edges) {
        if (dsu.unite(e.u, e.v)) total += e.w;
    }
    return total;
}
```

### Prim’s Algorithm

- เริ่มจาก node ใดก็ได้
- ค่อย ๆ ขยาย partial MST
- ทุกครั้งเลือก edge ที่เบาสุดที่เชื่อมจากต้นไม้เดิมไปยัง node ใหม่
- มองคล้าย Dijkstra แต่เป้าหมายคือ MST ไม่ใช่ shortest path

time complexity:
- ถ้าใช้ priority queue และ adjacency list มักเป็น `O((n + e) log n)`
- ถ้าใช้ adjacency matrix แบบตรงไปตรงมา มักเป็น `O(n^2)`

โค้ดตัวอย่าง (C++):
```cpp
#include <queue>
#include <utility>
#include <vector>

using WeightedEdge = std::pair<int, int>; // {to, weight}

int prim(int start, const std::vector<std::vector<WeightedEdge>>& adj) {
    int n = adj.size();
    std::vector<bool> inMST(n, false);
    std::priority_queue<
        std::pair<int, int>,
        std::vector<std::pair<int, int>>,
        std::greater<>
    > pq;

    pq.push({0, start});
    int totalWeight = 0;

    while (!pq.empty()) {
        auto [weight, u] = pq.top();
        pq.pop();

        if (inMST[u]) continue;
        inMST[u] = true;
        totalWeight += weight;

        for (auto [v, w] : adj[u]) {
            if (!inMST[v]) {
                pq.push({w, v});
            }
        }
    }

    return totalWeight;
}
```

## 7) สิ่งที่ต้องจำ

- `DFS` = ลึกก่อน
- `BFS` = ทีละชั้น และใช้กับ shortest pat
- h บน unweighted graph
- `connected component` หาได้จากการ traversal หลายรอบ
- `topological sort` ใช้กับ DAG เท่านั้น
- `Kruskal` เลือก edge เบาสุดทั่วกราฟ
- `Prim` ขยายต้นไม้จาก node ที่มีอยู่แล้ว
