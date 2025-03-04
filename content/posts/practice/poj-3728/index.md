+++
title = "POJ 3728 The merchant"
date = 2025-03-04T13:54:19+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3728

https://vjudge.net/problem/POJ-3728
<!--more-->
## 問題概要
- ある国には$N$個の都市があり、各都市のペアの間には1つの単純パスしかない。（都市の繋がりは木として表される）
- 物価が都市によって異なり、ある商品の都市$i$での値段は$w_i$である。

- $Q$個のクエリに答えよ
	- クエリ $a_i, b_i$ : 都市$a_i$から都市$b_i$へ移動する際、そのパス上のある都市で商品を購入し、それ以降のある都市で売却する。$\max(得られる利益の最大値, 0)$を出力せよ
### 制約
- $1\leq N,w_i,Q\leq5\times10^4$
## 解法メモ
- LCAをダブリングで求めるときに、一緒に$(その区間w_iの最大値, その区間のw_iの最小値,その区間内で得られる利益の最大値)$を分割統治で求める。これのマージは可換でないので左右からの計算結果を持つ必要がある。
- 計算量は準備に$O(N\log N)$, クエリあたり$O(\log N)$

HL分解＋セグメント木(or Disjoint Sparse Table)で実装する方がが楽そう
## 実装例
HL分解で実装したらTLEが取れなかったのでダブリング解法。TLギリギリなのであまりよい実装ではなさそう
```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <stack>
#include <vector>
typedef long long ll;
#define rep(i, n) for (ll i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
using namespace std;

struct S {
    int maxW, minW, maxVal;
    S(int maxW = 0, int minW = 0, int maxVal = 0) : maxW(maxW), minW(minW), maxVal(maxVal) {}
};
S e() {
    S res;
    res.maxW = -inf;
    res.minW = inf;
    res.maxVal = -inf;
    return res;
}
S op(S a, S b) {
    S res;
    res.maxW = max(a.maxW, b.maxW);
    res.minW = min(a.minW, b.minW);
    res.maxVal = max(b.maxW - a.minW, max(a.maxVal, b.maxVal));
    return res;
}

template <typename edge>
struct LCA {
   private:
    const vector<vector<edge> > &G;
    int root, logV;
    vector<vector<int> > _parent;
    vector<int> _depth;

    vector<vector<S> > data, dataRev;  // *

    void dfs() {
        _parent.at(0).at(root) = -1;
        _depth.at(root) = 0;
        stack<int, vector<int> > s;
        s.push(root);
        while (!s.empty()) {
            int cur = s.top();
            s.pop();
            for (int i = 0; i < G.at(cur).size(); i++) {
                const edge &e = G.at(cur).at(i);
                if (e.to != _parent.at(0).at(cur)) {
                    _parent.at(0).at(e.to) = cur;
                    _depth.at(e.to) = _depth.at(cur) + 1;
                    s.push(e.to);
                }
            }
        }
    }

   public:
    LCA(const vector<vector<edge> > &G, const vector<S> &w) : G(G), root(0) {
        logV = 1;
        int _v = 1;
        while (_v < int(G.size())) {
            _v *= 2;
            logV++;
        }
        _parent = vector<vector<int> >(logV, vector<int>(G.size()));
        _depth = vector<int>(G.size());
        dfs();

        data = vector<vector<S> >(logV, vector<S>(G.size(), e()));
        dataRev = vector<vector<S> >(logV, vector<S>(G.size(), e()));
        rep(i, G.size()) {
            data[0][i] = w[i];
            dataRev[0][i] = w[i];
        }

        for (int k = 0; k < logV - 1; k++) {
            for (int v = 0; v < int(G.size()); v++) {
                if (_parent.at(k).at(v) < 0) {
                    _parent.at(k + 1).at(v) = -1;
                    data[k + 1][v] = e();
                    dataRev[k + 1][v] = e();
                } else {
                    _parent.at(k + 1).at(v) = _parent.at(k).at(_parent.at(k).at(v));
                    data[k + 1][v] = op(data[k][v], data[k][_parent[k][v]]);
                    dataRev[k + 1][v] = op(dataRev[k][_parent[k][v]], dataRev[k][v]);
                }
            }
        }
    }

    int query(int u, int v) {
        if (_depth.at(u) > _depth.at(v)) swap(u, v);
        for (int k = 0; k < logV; k++) {
            if ((_depth.at(v) - _depth.at(u)) >> k & 1) {
                v = _parent.at(k).at(v);
            }
        }
        if (u == v) return u;
        for (int k = logV - 1; k >= 0; k--) {
            if (_parent.at(k).at(u) != _parent.at(k).at(v)) {
                u = _parent.at(k).at(u);
                v = _parent.at(k).at(v);
            }
        }
        return _parent.at(0).at(u);
    }

    S prod(int u, int v) {
        bool isRev = false;
        if (_depth.at(u) > _depth.at(v)) {
            swap(u, v);
            isRev = true;
        }
        S resHead = e(), resTail = e();
        for (int k = 0; k < logV; k++) {
            if ((_depth.at(v) - _depth.at(u)) >> k & 1) {
                if (isRev) resHead = op(resHead, data[k][v]);
                else resTail = op(dataRev[k][v], resTail);
                v = _parent.at(k).at(v);
            }
        }

        if (u == v) {
            if (isRev) return op(resHead, data[0][u]);
            else return op(data[0][u], resTail);
        }

        for (int k = logV - 1; k >= 0; k--) {
            if (_parent.at(k).at(u) != _parent.at(k).at(v)) {
                if (isRev) {
                    resHead = op(resHead, data[k][v]);
                    resTail = op(dataRev[k][u], resTail);
                } else {
                    resHead = op(resHead, data[k][u]);
                    resTail = op(dataRev[k][v], resTail);
                }
                u = _parent.at(k).at(u);
                v = _parent.at(k).at(v);
            }
        }

        if (isRev) {
            resHead = op(resHead, data[0][v]);
            resTail = op(data[0][u], resTail);
        } else {
            resHead = op(resHead, data[0][u]);
            resTail = op(dataRev[0][v], resTail);
        }
        return op(resHead, op(data[0][_parent.at(0).at(u)], resTail));
    }
    const vector<vector<int> > &parent() { return _parent; }
    const vector<int> &depth() { return _depth; }
};

struct edge {
    int to;
    edge(int to) : to(to) {}
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n;
    cin >> n;
    vector<S> w(n);
    rep(i, n) {
        int ww;
        cin >> ww;
        w[i] = S(ww, ww, 0);
    }

    vector<vector<edge> > G(n);
    rep(i, n - 1) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        G[u].push_back(edge(v));
        G[v].push_back(edge(u));
    }

    LCA<edge> lca(G, w);

    int q;
    cin >> q;
    rep(i, q) {
        int u, v;
        cin >> u >> v;
        u--, v--;
        cout << lca.prod(u, v).maxVal << endl;
    }
}
```
