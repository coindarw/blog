+++
title = "POJ 2723 Get Luffy Out"
date = 2025-03-04T13:53:48+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2723

https://vjudge.net/problem/POJ-2723
<!--more-->
## 問題概要
- $2N$種類の鍵があり、$0, 1, 2, \cdots, 2N-1$の番号が付いている。これは$N$個のペアに分割されており、ペアの片方のみ選んで持ち込むことができる。
- $M$階の牢獄があり、$i$階($1\leq i \leq M$)に入るには扉を開ける必要がある。それぞれの扉には鍵穴が2つあり、$A_i,B_i$いずれか一方の鍵さえあれば開けることができる。
- 最大でいくつのドアを開けられるか

- なお、階のショートカットはできない（1,3階の扉が開けられても、2階の扉は開けられないなら答えは2つ）
### 制約
- $1\leq N\leq 2^{10}$
- $1\leq M\leq 2^{11}$
## 解法メモ
- 「$x$階までの扉を全て開けられるか？」を二分探索
- これは2-SATでできる

- 具体的には、各キーを持っていくかどうかをTrue/Falseで表す。
- キーの各ペアについてどちらかがFalseである必要がある
- 各扉についてはorがTrueである必要がある
## 実装例
TLが厳しいので、普段使っているSCCクラスで余計な計算（強連結成分を潰したDAGを求めるなど）をしていると通らなかった
```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <iostream>
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

    vector<int> _component;
    // vector<vector<int> > _scc;
    // vector<vector<int> > _dag;

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
        built = true;
    }
    // const vector<vector<int> >& scc() const {
    //     assert(built);
    //     return _scc;
    // }
    const vector<int>& component() const {
        assert(built);
        return _component;
    }
    // const vector<vector<int> >& dag() const {
    //     assert(built);
    //     return _dag;
    // }
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

int n, m;
vector<pair<int, int> > pairs;
vector<pair<int, int> > doors;
bool cmp(int x) {
    twosat ts(n * 2);
    rep(i, n) {
        int a = pairs[i].first, b = pairs[i].second;
        ts.add_clause(a, 0, b, 0);
    }
    rep(i, x) {
        int a = doors[i].first, b = doors[i].second;
        ts.add_clause(a, 1, b, 1);
    }
    return ts.satisfiable();
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        cin >> n >> m;
        if (n == 0 && m == 0) break;

        pairs = vector<pair<int, int> >(n);
        doors = vector<pair<int, int> >(m);
        rep(i, n) {
            int a, b;
            cin >> a >> b;
            pairs[i] = make_pair(a, b);
        }
        rep(i, m) {
            int a, b;
            cin >> a >> b;
            doors[i] = make_pair(a, b);
        }

        int ok = 0, ng = m + 1;
        while (abs(ng - ok) > 1) {
            int mid = (ng + ok) / 2;
            if (cmp(mid)) ok = mid;
            else ng = mid;
        }

        cout << ok << "\n";
    }
}
```