+++
title = "POJ 3678 Katu Puzzle"
date = 2025-03-04T13:54:14+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3678

https://vjudge.net/problem/POJ-3678
<!--more-->
## 問題概要
- 有向グラフ$G(V,E)$で表されるパズルが解けるかどうかを判定する。
- 各有向辺には$op \in \{\mathrm{AND},\mathrm{OR},\mathrm{XOR}\}, c\in\{0,1\}$がラベル付けされている。
- 「パズルが解ける」とは各頂点$i$に対して値$X_i$を書き込んだとき、すべての辺$(a,b)\in E$について$X_a\ op\ X_b=c$が成り立つような$X_i$の割り当てが存在することを指す。
### 制約
- $1\leq N\leq  10^3$
- $1\leq M\leq  10^6$

## 解法メモ
2-SATを使う。以下、AND,OR,XORは$\land,\lor,\oplus$で表す。


2-SATは$(a\lor b)\land(\lnot c\lor d)\land(e\lor ¬f)\land\cdots$ のように乗法標準形の割り当てができるので、この形にうまく変形する必要がある
- $X_a \lor X_b = 1$
	- そのまま
- $X_a \lor X_b = 0$
	- $(\lnot X_a \lor\lnot  X_a) \land (\lnot X_b \lor\lnot  X_b)=1$
- $X_a \land X_b = 1$
	- $(X_a \lor X_a) \land (X_b \lor X_b)=1$
- $X_a \land X_b = 0$
	- $(\lnot X_a \lor \lnot X_b)=1$
- $X_a \oplus X_b = 1$
	- $(X_a \lor X_b)\land(\lnot X_a \lor \lnot X_b)=1$
- $X_a \oplus X_b = 0$
	- $(\lnot X_a \lor X_b) \land (X_a \lor\lnot  X_b)=1$

```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <iostream>
#include <string>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

struct SCC {
   private:
    int v;
    vector<vector<int> > G;
    vector<vector<int> > rG;
    vector<int> vs;
    vector<bool> used;
    bool built;

    void dfs(int u) {
        used[u] = true;
        for (int i = 0; i < G[u].size(); ++i) {
            int to = G[u][i];
            if (!used[to]) {
                dfs(to);
            }
        }
        vs.push_back(u);
    }

    void rdfs(int u, int k) {
        used[u] = true;
        _component[u] = k;
        for (int i = 0; i < rG[u].size(); ++i) {
            int to = rG[u][i];
            if (!used[to]) rdfs(to, k);
        }
    }

    vector<vector<int> > _scc;
    vector<int> _component;
    vector<vector<int> > _dag;

   public:
    void add_edge(int from, int to) {
        G[from].push_back(to);
        rG[to].push_back(from);
    }

    SCC() : v(0), built(false) {}
    SCC(int v) : v(v), built(false) {
        G = vector<vector<int> >(v);
        rG = vector<vector<int> >(v);
        used = vector<bool>(v);
        _component = vector<int>(v);
    }

    void build() {
        if (built) return;
        fill(used.begin(), used.end(), false);
        vs.clear();
        for (int u = 0; u < v; ++u)
            if (!used[u]) dfs(u);
        fill(used.begin(), used.end(), false);
        int k = 0;
        for (int i = int(vs.size() - 1); i >= 0; --i)
            if (!used[vs[i]]) rdfs(vs[i], k++);
        _scc = vector<vector<int> >(k);
        for (int u = 0; u < v; ++u) _scc[_component[u]].push_back(u);
        vector<pair<int, int> > edges;
        for (int u = 0; u < v; ++u)
            for (int i = 0; i < G[u].size(); ++i) {
                int to = G[u][i];
                if (_component[u] != _component[to]) edges.push_back(make_pair(_component[u], _component[to]));
            }
        sort(edges.begin(), edges.end());
        edges.erase(unique(edges.begin(), edges.end()), edges.end());
        _dag = vector<vector<int> >(k);
        for (int i = 0; i < edges.size(); ++i) {
            int from = edges[i].first;
            int to = edges[i].second;
            _dag[from].push_back(to);
        }
        built = true;
    }
    const vector<vector<int> >& scc() const {
        assert(built);
        return _scc;
    }
    const vector<int>& component() const {
        assert(built);
        return _component;
    }
    const vector<vector<int> >& dag() const {
        assert(built);
        return _dag;
    }
};
struct twosat {
   private:
    int n;
    bool built;
    vector<bool> _answer;
    SCC scc;

   public:
    twosat() : built(false) {}
    twosat(int n) : n(n), built(false), scc(2 * n) {}
    void add_clause(int i, bool f, int j, bool g) {
        assert(0 <= i && i < n);
        assert(0 <= j && j < n);
        scc.add_edge(2 * i + int(!f), 2 * j + int(g));
        scc.add_edge(2 * j + int(!g), 2 * i + int(f));
    }
    void add_clause(int i, bool f) { add_clause(i, f, i, f); }
    bool satisfiable() {
        built = true;
        scc.build();
        _answer.resize(n);
        for (int i = 0; i < n; i++) {
            if (scc.component()[2 * i] == scc.component()[2 * i + 1]) return false;
            _answer[i] = scc.component()[2 * i] < scc.component()[2 * i + 1];
        }
        return true;
    }
    vector<bool> answer() {
        assert(built);
        return _answer;
    }
};
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, m;
    cin >> n >> m;
    twosat ts(n);

    rep(i, m) {
        int a, b, c;
        string op;
        cin >> a >> b >> c >> op;
        if (op == "OR") {
            if (c == 1) {
                ts.add_clause(a, 1, b, 1);
            } else {
                ts.add_clause(a, 0);
                ts.add_clause(b, 0);
            }
        } else if (op == "AND") {
            if (c == 1) {
                ts.add_clause(a, 1);
                ts.add_clause(b, 1);
            } else {
                ts.add_clause(a, 0, b, 0);
            }
        } else if (op == "XOR") {
            if (c == 1) {
                ts.add_clause(a, 1, b, 1);
                ts.add_clause(a, 0, b, 0);
            } else {
                ts.add_clause(a, 1, b, 0);
                ts.add_clause(a, 0, b, 1);
            }
        }
    }

    cout << (ts.satisfiable() ? "YES" : "NO") << "\n";
}
```

