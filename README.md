# Graph Theory - Coding Project

**Group 7:**
| Name | NRP | Class |
| ---- | --- | ----- |
| Nuha Usama Okbah | 5025241005 | IUP |
| Embun Nabila Rasendriya Az Zahra  | 5025241009 | IUP |
| Almira Nayla Felisitha  | 5025241014 | IUP |
| Adelia Tanalina Yumna  | 5025241078 | IUP |



## 1. Travelling Salesman Problem  
**A. Code**
```c
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n, e;
    cin >> n >> e;

    vector<vector<int>> dist(n, vector<int>(n, 1e9));
    vector<vector<int>> edgeID(n, vector<int>(n, -1));

    for (int i = 0; i < e; i++) {
        int id, u, v, w;
        cin >> id >> u >> v >> w;
        u--; v--;
        if (w < dist[u][v]) {  
            dist[u][v] = dist[v][u] = w;
            edgeID[u][v] = edgeID[v][u] = id;
        }
    }

    int start;
    cin >> start;
    start--;

    vector<int> nodes;
    for (int i = 0; i < n; i++) {
        if (i != start) nodes.push_back(i);
    }

    int bestCost = 1e9;
    vector<int> bestEdges;

    do {
        int cost = 0;
        int curr = start;
        vector<int> edges;

        for (int nxt : nodes) {
            cost += dist[curr][nxt];
            edges.push_back(edgeID[curr][nxt]);
            curr = nxt;
        }

        cost += dist[curr][start];
        edges.push_back(edgeID[curr][start]);

        if (cost < bestCost) {
            bestCost = cost;
            bestEdges = edges;
        }

    } while (next_permutation(nodes.begin(), nodes.end()));

    cout << "Cost: " << bestCost << "\nRoute: ";
    for (int i = 0; i < (int)bestEdges.size(); i++) {
        cout << bestEdges[i];
        if (i != (int)bestEdges.size() - 1) cout << ", ";
    }
    cout << "\n";
}
```
**B. Explanation**

1) Read sizes and prepare matrices
```
int n, e;
cin >> n >> e;

vector<vector<int>> dist(n, vector<int>(n, 1e9));
vector<vector<int>> edgeID(n, vector<int>(n, -1));

```
- n = number of vertices, e = number of edges.
- dist is an n×n matrix initialized to a large value (1e9) to mean “no direct edge.”
- edgeID stores the input ID of the chosen edge between any pair

2) Load edges
```c
for (int i = 0; i < e; i++) {
    int id, u, v, w;
    cin >> id >> u >> v >> w;
    u--; v--;
    if (w < dist[u][v]) {  
        dist[u][v] = dist[v][u] = w;
        edgeID[u][v] = edgeID[v][u] = id;
    }
}

``` 
- Input gives id, endpoints u and v (1-based), and weight w. Convert to 0-based.
- If multiple edges connect the same pair, only the smallest weight is kept in dist, and its id is recorded in edgeID.
- Graph is undirected, so assign both [u][v] and [v][u].

3) Read the fixed start city
```c
int start;
cin >> start;
start--;

```
- The tour must start (and later return) at this vertex.
- Convert to 0-based to match matrix indices.

4) Build the list of cities to permute
```c
vector<int> nodes;
for (int i = 0; i < n; i++) {
    if (i != start) nodes.push_back(i);
}
```
- Generate all permutations of the non-start vertices, forming Hamiltonian tours that start at start, visit each other city exactly once, then return to start.

   
5) Track the best tour found so far
```c
int bestCost = 1e9;
vector<int> bestEdges;

```
- bestCost holds the minimum total cost discovered.
- bestEdges stores the sequence of edge IDs along that best tour (including the final edge back to start).

6) Enumerate all permutations and evaluate each tour
```c
do {
    int cost = 0;
    int curr = start;
    vector<int> edges;

    for (int nxt : nodes) {
        cost += dist[curr][nxt];
        edges.push_back(edgeID[curr][nxt]);
        curr = nxt;
    }

    cost += dist[curr][start];
    edges.push_back(edgeID[curr][start]);

    if (cost < bestCost) {
        bestCost = cost;
        bestEdges = edges;
    }

} while (next_permutation(nodes.begin(), nodes.end()));

```
- next_permutation iterates through every ordering of nodes
- For the current ordering, simulate the path 
- Sum weights from dist and record each traversed edge’s ID from edgeID.
- After closing the cycle (return to start), compare the total with bestCost

7) Print the minimal cost and the chosen edge IDs in order
```c
cout << "Cost: " << bestCost << "\nRoute: ";
for (int i = 0; i < (int)bestEdges.size(); i++) {
    cout << bestEdges[i];
    if (i != (int)bestEdges.size() - 1) cout << ", ";
}
cout << "\n";
```
- Print the minimum tour cost and the comma-separated list of edge IDs along that tour, including the final return edge


**C. Input-Output Samples**

Input
```
3 
4 
0 1 2 10 
1 2 3 5
2 3 1 7 
3 3 1 2 
1
```

Output
```
Cost: 17
Route: 0, 1, 3
```

## 2. Chinese Postman Problem

**A. Code**
```c
#include <iostream>
#include <vector>
#include <limits>
#include <algorithm>
#include <utility>
#include <map>
#include <numeric>   // for std::accumulate

using namespace std;

struct Edge { int u, v, w; };

int main() {
    int n, e;
    if (!(cin >> n)) return 0;
    cin >> e;

    const int INF = 1e9;

    vector<Edge> edges(e);
    vector<vector<int>> dist(n, vector<int>(n, INF));
    vector<vector<int>> nxt(n, vector<int>(n, -1));
    vector<int> degree(n, 0);
    map<pair<int,int>, int> bestId;

    for (int i = 0; i < n; ++i) dist[i][i] = 0;

    for (int i = 0; i < e; ++i) {
        int id, u, v, w;
        cin >> id >> u >> v >> w;
        --u; --v;
        edges[id] = {u, v, w};

        if (w < dist[u][v]) {
            dist[u][v] = dist[v][u] = w;
            nxt[u][v] = v;
            nxt[v][u] = u;
            bestId[make_pair(min(u,v), max(u,v))] = id;
        } else if (w == dist[u][v]) {
            pair<int,int> key = make_pair(min(u,v), max(u,v));
            if (!bestId.count(key) || id < bestId[key]) bestId[key] = id;
        }

        degree[u]++; degree[v]++;
    }

    int start; cin >> start; --start;

    // Floyd–Warshall
    for (int k = 0; k < n; ++k)
        for (int i = 0; i < n; ++i)
            if (dist[i][k] < INF)
                for (int j = 0; j < n; ++j)
                    if (dist[k][j] < INF && dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                        nxt[i][j] = nxt[i][k];
                    }

    vector<int> odd;
    for (int i = 0; i < n; ++i) if (degree[i] % 2) odd.push_back(i);

    // multiplicity tiap edge (mulai 1)
    vector<int> mult(e, 1);

    // pair odd dengan bitmask DP
    if (!odd.empty()) {
        int m = (int)odd.size();
        vector<int> dp(1 << m, INF), pre(1 << m, -1);
        dp[0] = 0;

        for (int mask = 0; mask < (1 << m); ++mask) {
            if (dp[mask] == INF) continue;
            int i = 0;
            while (i < m && (mask & (1 << i))) ++i;
            if (i == m) continue;
            for (int j = i + 1; j < m; ++j) if (!(mask & (1 << j))) {
                int a = odd[i], b = odd[j];
                int nmask = mask | (1 << i) | (1 << j);
                int cand = dp[mask] + dist[a][b];
                if (cand < dp[nmask]) {
                    dp[nmask] = cand;
                    pre[nmask] = (i << 8) | j;
                }
            }
        }

        // reconstruct pairings -> tambah mult pada path terpendek
        int mask = (1 << m) - 1;
        while (mask) {
            int ij = pre[mask];
            int i = (ij >> 8) & 0xFF;
            int j = ij & 0xFF;
            int a = odd[i], b = odd[j];

            int cur = a;
            while (cur != b) {
                int nx = nxt[cur][b];
                int id = bestId[make_pair(min(cur,nx), max(cur,nx))];
                mult[id]++; // duplikasi
                cur = nx;
            }

            mask ^= (1 << i);
            mask ^= (1 << j);
        }
    }

    // bangun multigraph untuk Hierholzer: (to, edge_id)
    vector<vector<pair<int,int>>> adj(n);
    vector<int> remaining(e, 0);
    for (int id = 0; id < e; ++id) {
        remaining[id] = mult[id];
        for (int k = 0; k < mult[id]; ++k) {
            adj[edges[id].u].push_back(make_pair(edges[id].v, id));
            adj[edges[id].v].push_back(make_pair(edges[id].u, id));
        }
    }

    // Hierholzer dari start
    vector<int> route;
    route.reserve(accumulate(mult.begin(), mult.end(), 0));
    vector<int> it(n, 0), st, edgeStack;
    st.reserve(route.capacity());
    edgeStack.reserve(route.capacity());

    st.push_back(start);
    edgeStack.push_back(-1);

    while (!st.empty()) {
        int u = st.back();
        vector<pair<int,int>>& lst = adj[u];
        while (it[u] < (int)lst.size() && remaining[lst[it[u]].second] == 0) ++it[u];
        if (it[u] == (int)lst.size()) {
            if (edgeStack.back() != -1) route.push_back(edgeStack.back());
            st.pop_back();
            edgeStack.pop_back();
        } else {
            pair<int,int> p = lst[it[u]];
            ++it[u];
            int v = p.first;
            int id = p.second;
            if (remaining[id] == 0) continue;
            remaining[id]--;
            st.push_back(v);
            edgeStack.push_back(id);
        }
    }
    reverse(route.begin(), route.end());

    long long cost = 0;
    for (int id = 0; id < e; ++id) cost += 1LL * edges[id].w * mult[id];

    cout << "Cost: " << cost << "\n";
    cout << "Route: ";
    for (size_t i = 0; i < route.size(); ++i) {
        cout << route[i];
        if (i + 1 < route.size()) cout << ", ";
    }
    cout << "\n";
    return 0;
}
```

**B. Explanation**
1) Input
    ```c
    cin >> n >> e;
    vector<Edge> edges(e);
    vector<vector<int>> dist(n, vector<int>(n, INF));
    vector<int> degree(n, 0);

    ```
    - Creates storage for edges, distances, and vertex degrees.
    - `dist[i][i] = 0` (no cost to go to itself).

2) Find shortest path
    ```c
    for (int k = 0; k < n; ++k)
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j)
            if (dist[i][k] + dist[k][j] < dist[i][j]) {
                dist[i][j] = dist[i][k] + dist[k][j];
                nxt[i][j] = nxt[i][k];
            }

    ```
    - This computes all-pairs shortest paths between every pair of vertices.
    - `nxt[i][j]` lets the program later reconstruct the actual path between two vertices.

3) Find odd-degree vertices
    ```c
    vector<int> odd;
    for (int i = 0; i < n; ++i)
        if (degree[i] % 2) odd.push_back(i);
    ```
    - If some vertices have odd degree, the graph isn’t Eulerian — those are the ones you must “pair up” by duplicating paths between them.

4) Pair up odd vertices
    ```c
    dp[0] = 0;
    for (mask = 0; mask < (1 << m); ++mask)
    ```
    - `dp[mask]` = minimal extra cost for pairing the vertices represented in `mask`
    - `pre[mask]` = record of which pair (i,j) was chosen
    - `dp[(1 << m) - 1]` gives minimal added cost.

5) Add duplicate edges
    ```c
    int id = bestId[{min(cur,nx), max(cur,nx)}];
    mult[id]++;
    ```
    - After DP, the algorithm reconstructs which odd pairs should be connected.
    - It follows the shortest path between them and increases the multiplicity (`mult`) of each edge used in that path — effectively duplicating those edges to make degrees even.

6) Build adjacency list for Euler circuit
    ```c
    for (int id = 0; id < e; ++id) {
        for (int k = 0; k < mult[id]; ++k) {
            adj[u].push_back({v, id});
            adj[v].push_back({u, id});
        }
    }
    ```
    - Builds a multigraph that includes all edges (original + duplicates).
    - Each edge is added as many times as needed (`mult[id]` times), so the new graph is Eulerian.

7) Find the route
    ```c
    st.push_back(start);
    while (!st.empty()) { ... }
    ```
    - Finds a path that uses every edge exactly once (counting duplicates).

8) Calculate total cost
    ```c
    for (int id = 0; id < e; ++id)
    cost += edges[id].w * mult[id];
    ```
    - Sum of each edge’s weight times how many times it is used (including duplicates added earlier).

9) Print results
    ```c
    cout << "Cost: " << cost << "\n";
    cout << "Route: " << route;
    ```
    - Give the result of final cost and route. 

**C. Input-Output Samples**
Input
```
3 
4 
0 1 2 10 
1 2 3 5
2 3 1 7 
3 3 1 2 
1
```

Output
```
Cost: 17
Route: 0, 1, 3
```

<img width="682" height="329" alt="Screenshot 2025-10-07 195406" src="https://github.com/user-attachments/assets/f0ce3540-6d09-47be-b395-f2b29444bba5" />



    
