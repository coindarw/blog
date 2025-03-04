+++
title = "POJ 3180 The Cow Prom(luogu P2863)"
date = 2025-03-04T13:53:57+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3180
https://www.luogu.com.cn/problem/P2863

https://vjudge.net/problem/POJ-3180
https://vjudge.net/problem/%E6%B4%9B%E8%B0%B7-P2863
（海外のカートゥーンなど出てくる（？）、牧場で馬が水を飲んでいたりする丸いタンクを[ストック・タンク](https://ja.wikipedia.org/wiki/%E3%82%B9%E3%83%88%E3%83%83%E3%82%AF%E3%83%BB%E3%82%BF%E3%83%B3%E3%82%AF)というらしい ）
<!--more-->
## 問題概要
- $N$頭の牛がいる。$1,2,\cdots,N$の番号が付けられており、ストック・タンクを囲んで時計回りに円形に並んでいる。
- $M$本のロープがあり、$i$番目のロープは牛$A_i$と牛$B_i$をその方向に時計回りに繋いでいる。
- 牛を頂点、ロープを有向辺とみなしたグラフのサイズ2以上の強連結成分はいくつあるか

### 制約
- $2\leq N\leq 10^4$
- $2\leq M\leq 5\times10^4$

## 解法メモ
読解が難しい。強連結成分の問題だと知らなかったら何を出力すればいいのかわからなかったと思う。

解法としてはSCCするだけ
## 実装例
範囲forなどC++98用に少し書き換えている。
language:C++なら適当でも通るがG++だと `ios_base::sync_with_stdio(false), cin.tie(NULL)` 無しだとTLEする。
```cpp
#include <algorithm>
#include <cassert>
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
        // for (int to : G[u]) {
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
            // for (int to : G[u])
            for (int i = 0; i < G[u].size(); ++i) {
                int to = G[u][i];
                if (_component[u] != _component[to]) edges.push_back(make_pair(_component[u], _component[to]));
            }
        sort(edges.begin(), edges.end());
        edges.erase(unique(edges.begin(), edges.end()), edges.end());
        _dag = vector<vector<int> >(k);
        // for (auto [from, to] : edges) _dag[from].push_back(to);
        for (int i = 0; i < edges.size(); ++i) {
            int from = edges[i].first;
            int to = edges[i].second;
            _dag[from].push_back(to);
        }
        built = true;
    }
    vector<vector<int> > scc() const {
        assert(built);
        return _scc;
    }
    vector<int> component() const {
        assert(built);
        return _component;
    }
    vector<vector<int> > dag() const {
        assert(built);
        return _dag;
    }
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, m;
    cin >> n >> m;
    SCC scc(n);
    rep(i, m) {
        int a, b;
        cin >> a >> b;
        a--, b--;
        scc.add_edge(a, b);
    }
    scc.build();
    vector<vector<int> > res = scc.scc();

    int ans = 0;
    rep(i, res.size()) {
        if (res[i].size() > 1) ans++;
    }
    cout << ans << '\n';
}
```