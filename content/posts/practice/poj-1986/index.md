+++
title = "POJ 1986 Distance Queries"
date = 2025-03-04T13:53:28+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1986
http://bailian.openjudge.cn/practice/1986?lang=en_US

https://vjudge.net/problem/POJ-1986
https://vjudge.net/problem/OpenJ_Bailian-1986


POJだとMLがきついのでOpenJ_BailianでAC。
<!--more-->
## 問題概要
（ [POJ 1984 Navigation Nightmare](http://poj.org/problem?id=1984)と同じ入力形式の問題なのでこちらの問題文を読む必要もある ）

- 2次元平面状に$N$個の農場があり、$1\cdots N$の番号が付いている
- 長さの異なる$M$本の道路があり、$i$本目の道路は$A_i$番目の農場から$B_i$番目の農場へ続く距離$L_i$の道路で、$D_i\in\{N,E,W,S\}$方向につながっている。
- 2つの農場の間にはちょうど1つパスが存在する。（木になっている)

- 面倒な入力は与えられない。
	- 農場は道路の端点にのみある
	- 道路が交差することはない
	- 矛盾するデータが来ることはない

- $K$個のクエリが与えられるので答えよ。
- クエリ $a_i,b_i$ : 農場$a_i$から農場$b_i$に移動するとき通る道路の長さの合計を求めよ
### 制約
- $2\leq N\leq 4\times10^4$
- $1\leq M< 4\times10^4$
- $1\leq L_i\leq 10^3$
- $1\leq K\leq 10^4$

## 解法メモ
- 木の適当な頂点を根とする。
- 根から他の頂点$v$への距離$d(v)$を列挙しておけば、答えは$d(a_i)+d(b_i)-2d(\mathrm{lca}(a_i,b_i))$となる。
- 方位が与えられるがこの問題では特に関係ない
## 実装例
POJだとMLが30000 kBなのでMLE
```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <iostream>
#include <stack>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
using namespace std;
#define all(v) v.begin(), v.end()

template <typename edge>
struct LCA {
   private:
    class SparseTable {
        vector<vector<unsigned long long> > st;
        vector<int> lookup;
        int n, N, logN;

       public:
        SparseTable() {}
        SparseTable(const vector<unsigned long long> &v) {
            n = v.size();
            if (n == 0) return;
            N = 1 << (32 - __builtin_clz(v.size()));
            logN = 32 - __builtin_clz(N);
            st.assign(logN, vector<unsigned long long>(N, (unsigned long long)-1));
            for (int i = 0; i < n; ++i) st[0][i] = v[i];
            int len = 1;
            for (int i = 1; i < (int)st.size(); ++i) {
                for (int j = 0; j + len < N; j++) {
                    st[i][j] = min(st[i - 1][j], st[i - 1][j + len]);
                }
                len <<= 1;
            }
            lookup.resize(N + 1);
            for (int i = 2; i < (int)lookup.size(); ++i) lookup[i] = lookup[i >> 1] + 1;
        }

        unsigned long long prod(int l, int r) {
            assert(0 <= l && l <= r && r <= n);
            if (l == r) return (unsigned long long)-1;
            int b = lookup[r - l];
            return min(st[b][l], st[b][r - (1 << b)]);
        }
    };
    const vector<vector<edge> > &G;
    SparseTable st;

    void dfs() {
        stack<int, vector<int> > s;
        s.push(root);
        while (!s.empty()) {
            int v = s.top();
            s.pop();
            if (v >= 0) {
                _id[v] = _vs.size();
                _vs.push_back(v);
                for (int i = 0; i < int(G[v].size()); ++i) {
                    const edge &e = G[v][i];
                    if (_parent[v] == e.to) continue;
                    _parent[e.to] = v;
                    _depth[e.to] = _depth[v] + 1;
                    s.push(~v);
                    s.push(e.to);
                }
            } else {
                _vs.push_back(~v);
            }
        }
    }

   public:
    int root, logV;
    vector<int> _parent, _depth, _vs, _id;
    LCA(const vector<vector<edge> > &_G, int _root = 0) : G(_G), root(_root) {
        logV = 1;
        int _v = 1;
        while (_v < int(G.size())) _v *= 2, logV++;
        _parent = vector<int>(G.size());
        _depth = vector<int>(G.size());
        _id = vector<int>(G.size());
        dfs();
        vector<unsigned long long> v(_vs.size());
        for (int i = 0; i < int(_vs.size()); ++i) v[i] = ((unsigned long long)_depth[_vs[i]] << 32) | _vs[i];
        st = SparseTable(v);
    }

    int getLca(int u, int v) { return st.prod(min(_id[u], _id[v]), max(_id[u], _id[v]) + 1) & ((1ull << 32) - 1); }
    int getDist(int u, int v) { return _depth[u] + _depth[v] - 2 * _depth[getLca(u, v)]; }
};

struct edge {
    int to;
    int cost;
    edge(int _to, int _cost) : to(_to), cost(_cost) {}
};
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, m;
    cin >> n >> m;
    vector<vector<edge> > G(n);

    rep(i, m) {
        int a, b, c;
        char d;
        cin >> a >> b >> c >> d;
        a--, b--;
        G[a].push_back(edge(b, c));
        G[b].push_back(edge(a, c));
    }
    vector<int> dist(n, inf);
    dist[0] = 0;
    stack<int, vector<int> > st;
    st.push(0);
    while (!st.empty()) {
        int u = st.top();
        st.pop();
        rep(i, G[u].size()) {
            edge e = G[u][i];
            if (dist[e.to] > dist[u] + e.cost) {
                dist[e.to] = dist[u] + e.cost;
                st.push(e.to);
            }
        }
    }

    LCA<edge> lca(G);
    int q;
    cin >> q;
    rep(i, q) {
        int a, b;
        cin >> a >> b;
        a--, b--;
        cout << dist[a] + dist[b] - 2 * dist[lca.getLca(a, b)] << endl;
    }
}
```