# Algorithm Cheat Sheet

## ภาพรวม
ไฟล์นี้สรุป algorithm และ technique หลักจากทุกบทใน `algo_final` โดยแต่ละหัวข้อมีครบ:
- `description`
- `time complexity`
- `space complexity`
- `pros`
- `cons`
- `use case`
- `example code`

## 1) Fractional Knapsack

`description`
- เลือก item ตาม `value / weight ratio` จากมากไปน้อย
- ถ้าหยิบทั้งชิ้นไม่ได้ ให้หยิบเป็น fraction

`time complexity`
- `O(n log n)` จากการ sort

`space complexity`
- `O(1)` เพิ่มเติม ถ้า sort in-place
- ถ้าคิดรวม input เป็น `O(n)`

`pros`
- เร็ว
- ได้คำตอบ optimal สำหรับ `fractional knapsack`

`cons`
- ใช้ไม่ได้กับ `0-1 knapsack`

`use case`
- ปัญหาที่แบ่ง resource ได้ เช่น ตัดวัสดุ, แบ่งสินค้า, บรรจุของแบบแบ่งชิ้นได้

`example code`
```cpp
#include <algorithm>
#include <vector>
using namespace std;

struct Item {
    double weight, value;
};

double fractionalKnapsack(double capacity, vector<Item> items) {
    sort(items.begin(), items.end(), [](const Item& a, const Item& b) {
        return (a.value / a.weight) > (b.value / b.weight);
    });

    double ans = 0.0;
    for (const auto& item : items) {
        if (capacity <= 0) break;
        double take = min(capacity, item.weight);
        ans += take * (item.value / item.weight);
        capacity -= take;
    }
    return ans;
}
```

## 2) Activity Selection

`description`
- เรียงกิจกรรมตาม `finish time`
- เลือกกิจกรรมที่จบเร็วที่สุดก่อน
- จากนั้นเลือกกิจกรรมถัดไปที่ไม่ชนกับงานล่าสุด

`time complexity`
- `O(n log n)`

`space complexity`
- `O(1)` เพิ่มเติม ถ้า sort in-place

`pros`
- ง่าย
- ได้คำตอบ optimal สำหรับโจทย์เลือกกิจกรรมไม่ชนกัน

`cons`
- ใช้ไม่ได้กับโจทย์ scheduling ทั่วไปทุกแบบ

`use case`
- เลือกงาน, ห้องประชุม, ช่วงเวลาใช้งานทรัพยากรที่ไม่ overlap

`example code`
```cpp
#include <algorithm>
#include <vector>
using namespace std;

struct Activity {
    int start, finish;
};

vector<Activity> activitySelection(vector<Activity> jobs) {
    sort(jobs.begin(), jobs.end(), [](const Activity& a, const Activity& b) {
        return a.finish < b.finish;
    });

    vector<Activity> selected;
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

## 3) DFS

`description`
- เดินกราฟหรือ state space แบบ “ไปให้ลึกก่อน”
- ใช้ recursion หรือ stack

`time complexity`
- กราฟแบบ adjacency list: `O(n + e)`
- state space: `O(number of states visited)`

`space complexity`
- กราฟ: `O(n)`
- state space: `O(depth)`

`pros`
- ใช้ memory น้อยกว่า BFS
- เหมาะกับ recursion, backtracking, cycle detection, topological sort

`cons`
- ไม่ได้ให้ shortest path ใน unweighted graph
- ถ้า state space ใหญ่มากหรือไม่มีขอบเขต อาจลึกมากหรือวนไม่จบ

`use case`
- graph traversal
- connected components
- cycle detection
- topological sort
- backtracking

`example code`
```cpp
#include <vector>
using namespace std;

void dfs(int u, const vector<vector<int>>& adj, vector<bool>& visited) {
    visited[u] = true;
    for (int v : adj[u]) {
        if (!visited[v]) dfs(v, adj, visited);
    }
}
```

## 4) BFS

`description`
- เดินกราฟหรือ state space แบบ “ทีละชั้น”
- ใช้ queue

`time complexity`
- กราฟแบบ adjacency list: `O(n + e)`
- state space: `O(number of states visited)`

`space complexity`
- กราฟ: `O(n)`
- state space: อาจสูงถึงขนาด frontier ของชั้นนั้น

`pros`
- หา shortest path ได้ใน `unweighted graph`
- ได้คำตอบที่ใกล้ root ที่สุดใน search tree

`cons`
- ใช้ memory มากกว่า DFS
- ใช้ไม่ได้ตรง ๆ กับ weighted shortest path

`use case`
- shortest path ใน unweighted graph
- traversal แบบตามระยะ
- state space search ที่ต้องการคำตอบตื้นสุด

`example code`
```cpp
#include <queue>
#include <vector>
using namespace std;

vector<int> bfsDistance(int start, const vector<vector<int>>& adj) {
    vector<int> dist(adj.size(), -1);
    queue<int> q;
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

## 5) Topological Sort

`description`
- หาลำดับของ node ใน `DAG` ที่ทุก edge ชี้จากตัวก่อนหน้าไปตัวหลัง

`time complexity`
- `O(n + e)`

`space complexity`
- `O(n)`

`pros`
- ใช้จัดลำดับ dependency
- เป็นพื้นฐานของหลายโจทย์บน DAG

`cons`
- ใช้ได้เฉพาะ DAG
- ถ้ามี cycle จะทำไม่ได้

`use case`
- course scheduling
- build order
- dependency resolution

`example code`
```cpp
#include <algorithm>
#include <vector>
using namespace std;

void topoDfs(int u, const vector<vector<int>>& adj,
             vector<bool>& visited, vector<int>& order) {
    visited[u] = true;
    for (int v : adj[u]) {
        if (!visited[v]) topoDfs(v, adj, visited, order);
    }
    order.push_back(u);
}

vector<int> topologicalSort(const vector<vector<int>>& adj) {
    vector<bool> visited(adj.size(), false);
    vector<int> order;
    for (int i = 0; i < (int)adj.size(); ++i) {
        if (!visited[i]) topoDfs(i, adj, visited, order);
    }
    reverse(order.begin(), order.end());
    return order;
}
```

## 6) Kruskal’s Algorithm

`description`
- เรียง edges ตามน้ำหนัก
- เลือก edge ที่เบาสุดถัดไป ถ้ายังไม่ทำให้เกิด cycle
- ใช้ `DSU`

`time complexity`
- `O(e log e)` หรือ `O(e log n)`

`space complexity`
- `O(n)` สำหรับ DSU

`pros`
- เข้าใจง่าย
- ดีเมื่อสะดวกไล่ edges ทั้งหมด

`cons`
- ต้อง sort edges ก่อน
- ไม่ค่อยเหมาะถ้า graph โตมากและ stream มาเป็น node-by-node

`use case`
- minimum spanning tree
- graph connectivity with weighted edges

`example code`
```cpp
#include <algorithm>
#include <numeric>
#include <vector>
using namespace std;

struct Edge {
    int u, v, w;
};

struct DSU {
    vector<int> parent, sz;
    explicit DSU(int n) : parent(n), sz(n, 1) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        if (parent[x] == x) return x;
        return parent[x] = find(parent[x]);
    }
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;
        if (sz[a] < sz[b]) swap(a, b);
        parent[b] = a;
        sz[a] += sz[b];
        return true;
    }
};

int kruskal(int n, vector<Edge> edges) {
    sort(edges.begin(), edges.end(), [](const Edge& a, const Edge& b) {
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

## 7) Prim’s Algorithm

`description`
- เริ่มจาก node หนึ่ง
- ค่อย ๆ ขยาย MST โดยเลือก edge เบาสุดที่เชื่อมจาก tree เดิมไป node ใหม่

`time complexity`
- adjacency list + heap: `O((n + e) log n)`
- adjacency matrix: `O(n^2)`

`space complexity`
- `O(n + e)` ถ้ารวม adjacency list
- เพิ่มเติมจาก input มัก `O(n)`

`pros`
- เหมาะกับการขยาย tree ทีละ node
- ใช้ได้ดีเมื่อ graph เก็บเป็น adjacency list

`cons`
- ใช้กับ MST เท่านั้น
- ต้องระวัง graph ไม่ connected ถ้าต้องการ MST ครบทั้งกราฟ

`use case`
- minimum spanning tree
- network design

`example code`
```cpp
#include <queue>
#include <utility>
#include <vector>
using namespace std;

using WeightedEdge = pair<int, int>; // {to, weight}

int prim(int start, const vector<vector<WeightedEdge>>& adj) {
    int n = adj.size();
    vector<bool> inMST(n, false);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;

    pq.push({0, start});
    int totalWeight = 0;

    while (!pq.empty()) {
        auto [weight, u] = pq.top();
        pq.pop();
        if (inMST[u]) continue;
        inMST[u] = true;
        totalWeight += weight;

        for (auto [v, w] : adj[u]) {
            if (!inMST[v]) pq.push({w, v});
        }
    }
    return totalWeight;
}
```

## 8) Dijkstra’s Algorithm

`description`
- shortest path จาก source ไปทุก node
- ใช้ greedy + priority queue
- ใช้ได้เมื่อไม่มี negative edge

`time complexity`
- heap: `O((n + e) log n)`

`space complexity`
- `O(n)`

`pros`
- เร็วมากสำหรับ non-negative weighted graph
- เป็นมาตรฐานสำหรับ single-source shortest path

`cons`
- ใช้ไม่ได้กับ negative edge

`use case`
- routing
- map navigation
- weighted graph shortest path

`example code`
```cpp
#include <queue>
#include <utility>
#include <vector>
using namespace std;

using Edge = pair<int, int>; // {to, weight}

vector<int> dijkstra(int start, const vector<vector<Edge>>& adj) {
    const int INF = 1e9;
    vector<int> dist(adj.size(), INF);
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;

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

## 9) Bellman-Ford

`description`
- shortest path ที่รองรับ negative edge
- relax ทุก edge ซ้ำ `n - 1` รอบ
- ตรวจ negative cycle ได้

`time complexity`
- `O(n e)`

`space complexity`
- `O(n)`

`pros`
- รองรับ negative edge
- detect negative cycle ได้

`cons`
- ช้ากว่า Dijkstra มาก

`use case`
- weighted directed graph ที่มี negative edge
- ตรวจ negative cycle

`example code`
```cpp
#include <tuple>
#include <vector>
using namespace std;

struct BFResult {
    vector<int> dist;
    bool hasNegativeCycle;
};

BFResult bellmanFord(int n, int start, const vector<tuple<int, int, int>>& edges) {
    const int INF = 1e9;
    vector<int> dist(n, INF);
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

## 10) Floyd-Warshall

`description`
- shortest path สำหรับทุกคู่ node
- dynamic programming

`time complexity`
- `O(n^3)`

`space complexity`
- `O(n^2)`

`pros`
- โค้ดตรงไปตรงมา
- รองรับ negative edge
- ใช้ตรวจ negative cycle ได้

`cons`
- ช้ามากถ้า graph ใหญ่

`use case`
- all-pairs shortest path
- graph ขนาดไม่ใหญ่

`example code`
```cpp
#include <vector>
using namespace std;

void floydWarshall(vector<vector<int>>& dist, int INF) {
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

## 11) Backtracking

`description`
- search พร้อม prune โดยตัด partial state ที่ “รู้แล้วว่าไปต่อก็ไม่สำเร็จ”

`time complexity`
- worst case มัก exponential
- เช่น `O(2^n)` หรือ `O(n!)` แล้วแต่ search space

`space complexity`
- มักเป็น `O(depth)`

`pros`
- ลด search ได้มากเมื่อมี constraint แรง
- เขียนร่วมกับ recursion ได้ดี

`cons`
- worst case ยังช้ามาก
- ต้องออกแบบ pruning เอง

`use case`
- N-Queen
- subset sum
- permutation with constraints

`example code`
```cpp
#include <cmath>
#include <vector>
using namespace std;

bool safe(const vector<int>& col, int row, int c) {
    for (int r = 0; r < row; ++r) {
        if (col[r] == c) return false;
        if (abs(col[r] - c) == abs(r - row)) return false;
    }
    return true;
}

void solveQueens(int row, vector<int>& col, vector<vector<int>>& ans) {
    if (row == 8) {
        ans.push_back(col);
        return;
    }
    for (int c = 0; c < 8; ++c) {
        if (safe(col, row, c)) {
            col[row] = c;
            solveQueens(row + 1, col, ans);
        }
    }
}
```

## 12) Branch and Bound

`description`
- ใช้กับ optimization problem
- prune state ที่ต่อให้ดีที่สุดก็ยังแพ้คำตอบปัจจุบัน
- อาศัย `bounding heuristic`

`time complexity`
- worst case มัก exponential

`space complexity`
- ขึ้นกับ search strategy
- DFS-style มัก `O(depth)`
- best-first style มักสูงกว่า

`pros`
- ลดงานได้มากเมื่อ bound ดี
- เหมาะกับ optimization problem

`cons`
- ถ้า bound ไม่ดี ประสิทธิภาพจะตกมาก
- worst case ยังแย่

`use case`
- 0-1 Knapsack
- coin change แบบ optimization
- combinatorial optimization

`example code`
```cpp
#include <algorithm>
#include <vector>
using namespace std;

int bestAnswer = 0;
int W, n;
vector<int> w, p, tail;

void knapsackBnB(int step, int sumP, int sumW) {
    if (sumW > W) return;                        // backtracking
    if (step < n && sumP + tail[step] <= bestAnswer) return; // bound

    if (step == n) {
        bestAnswer = max(bestAnswer, sumP);
        return;
    }

    knapsackBnB(step + 1, sumP + p[step], sumW + w[step]);
    knapsackBnB(step + 1, sumP, sumW);
}
```

## 13) Best-First Search / Least-Cost Search

`description`
- เลือกขยาย state ที่ดูดีที่สุดก่อน
- มักใช้ priority queue

`time complexity`
- worst case มัก exponential
- มีค่า `log` เพิ่มจาก priority queue

`space complexity`
- มักสูง เพราะต้องเก็บ frontier จำนวนมาก

`pros`
- อาจเจอคำตอบดีเร็ว
- เหมาะกับ branch and bound แบบ prioritize state ที่ promising

`cons`
- ใช้ memory มาก
- ถ้า heuristic ไม่ดีอาจไม่ช่วย

`use case`
- branch and bound
- heuristic search

`example code`
```cpp
#include <queue>
#include <vector>
using namespace std;

struct State {
    int cost;
    int node;
    bool operator>(const State& other) const {
        return cost > other.cost;
    }
};

void bestFirstSearch(int start) {
    priority_queue<State, vector<State>, greater<State>> pq;
    pq.push({0, start});
    while (!pq.empty()) {
        State cur = pq.top();
        pq.pop();
        // expand cur
    }
}
```

## 14) Reduction

`description`
- ไม่ใช่ algorithm แก้โจทย์เดี่ยวโดยตรง แต่เป็น technique ในการแปลงปัญหา A ไปเป็นปัญหา B
- ใช้เปรียบเทียบ hardness ของปัญหา

`time complexity`
- ต้องเป็น polynomial time reduction

`space complexity`
- แล้วแต่ transformation ที่ใช้

`pros`
- สำคัญมากใน complexity theory
- ใช้พิสูจน์ NP-Hard / NP-Complete

`cons`
- ไม่ได้ใช้แก้โจทย์ปฏิบัติทุกวันโดยตรง

`use case`
- พิสูจน์ hardness
- แปลง problem formulation

`example code`
```cpp
#include <algorithm>
#include <vector>
using namespace std;

int kthSmallest(vector<int> a, int k) {
    sort(a.begin(), a.end()); // reduce kth smallest to sorting
    return a[k];
}
```

## 15) สรุปเลือกใช้เร็ว ๆ
- shortest path แบบไม่ติดลบ: `Dijkstra`
- shortest path แบบมี negative edge: `Bellman-Ford`
- shortest path ทุกคู่ node: `Floyd-Warshall`
- MST: `Kruskal` หรือ `Prim`
- unweighted shortest path / level traversal: `BFS`
- graph traversal / recursion-heavy search: `DFS`
- optimization with pruning: `Branch and Bound`
- constraint search: `Backtracking`
- greedy classic: `Fractional Knapsack`, `Activity Selection`
