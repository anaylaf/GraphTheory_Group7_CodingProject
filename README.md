# GraphTheory_Group7_CodingProject
## Chinese Postman Problem

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
    - The total cost is the sum of each edge’s weight times how many times it is used (including duplicates added earlier).

9) Print results
    ```c
    cout << "Cost: " << cost << "\n";
    cout << "Route: " << route;
    ```
    - Correct, just note that `route` prints edge IDs in traversal order, not vertex numbers.

**C. Input-Output Samples**

**Input**
```
4
5
0 1 2 3
1 2 3 4
2 3 4 5
3 4 1 2
4 1 3 1
1
```

**Output**
```
Cost: 16
Route: 0, 1, 2, 3, 4, 4
```

    
