+++
title = "POJ 2114 Boatherds(BOJ 6669,Gym 100811B)"
date = 2025-03-04T13:53:36+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2114
https://www.acmicpc.net/problem/6669
https://codeforces.com/gym/100811/attachments

https://vjudge.net/problem/POJ-2114
https://vjudge.net/problem/Baekjoon-6669
https://vjudge.net/problem/Gym-100811B
<!--more-->
## 問題概要
- ある場所の交通網は頂点数$N$の木として表され、各辺には通行料$c_i$が設定されている
- $M$個のクエリに答えよ
	- クエリ$i$：「通行料（パス上の辺の$c_i$の和）が$x_i$になる2頂点は存在するか」
### 制約
- $1\leq N\leq 10^4$
- $0\leq c_i\leq 1000$
- $M\leq 100$
- $1\leq x_i\leq 10^7$
## 解法メモ
以降、通行料を距離、根からの距離を深さと書く。

- 重心分解で解ける（初めて実装するのでコメント多め）
	- 重心分解を行うと、「パスが根を通る2頂点で距離が$x_i$になるものがあるか」だけ考えればよい
	- これは根からそれぞれの子方面に向けての深さのリストと、それらをまとめてソートしたリストを持っておき、二分探索等を使うことで求めることができる。
	- 同じ部分木の2頂点を間違って取らないよう注意
## 実装例
```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;
#define all(v) v.begin(), v.end()

int cnt = 0;
template <typename EDGE, void (*f)(int, const vector<bool> &, const vector<vector<EDGE> > &)>
struct CentroidDecomposition {
    const vector<vector<EDGE> > &G;
    vector<bool> centroid;
    vector<int> subtree_size;

    CentroidDecomposition(const vector<vector<EDGE> > &G) : G(G), centroid(G.size()), subtree_size(G.size()) {}

    void add_edge(int u, int v) {
        G[u].push_back(v);
        G[v].push_back(u);
    }

    void solve(int v = 0) {
        compute_subtree_size(v, -1);
        int s = search_centroid(v, -1, subtree_size[v]).second;
        centroid[s] = true;
        for (size_t i = 0; i < G[s].size(); ++i) {
            if (centroid[G[s][i].to]) continue;
            solve(G[s][i].to);
        }
        f(s, centroid, G);
        centroid[s] = false;
    }

   private:
    void compute_subtree_size(int v, int p) {
        subtree_size[v] = 1;
        for (size_t i = 0; i < G[v].size(); ++i) {
            const EDGE &e = G[v][i];
            if (e.to == p || centroid[e.to]) continue;
            compute_subtree_size(e.to, v);
            subtree_size[v] += subtree_size[e.to];
        }
    }

    pair<int, int> search_centroid(int v, int p, int t) {
        pair<int, int> res = make_pair<int, int>(G.size(), G.size());
        int sum = 1, m = 0;
        for (size_t i = 0; i < G[v].size(); ++i) {
            const EDGE &e = G[v][i];
            if (e.to == p || centroid[e.to]) continue;
            res = min(res, search_centroid(e.to, v, t));
            m = max(m, subtree_size[e.to]);
            sum += subtree_size[e.to];
        }
        m = max(m, t - sum);
        return min(res, make_pair(m, v));
    }
};

struct edge {
    int to;
    ll cost;
    edge(int _to, ll _cost) : to(_to), cost(_cost) {}
};

void enumerate_depths(int v, int p, const vector<bool> &centroid, const vector<vector<edge> > &G, vector<int> &depths, int cur = 0) {
    if (p == -1) depths.push_back(0);
    for (size_t i = 0; i < G[v].size(); ++i) {
        if (G[v][i].to == p || centroid[G[v][i].to]) continue;
        enumerate_depths(G[v][i].to, v, centroid, G, depths, cur + G[v][i].cost);
        depths.push_back(cur + G[v][i].cost);
    }
}

vector<int> xs;
vector<bool> found;
int found_count;
void solve(int v, const vector<bool> &centroid, const vector<vector<edge> > &G) {
    if (found_count == xs.size()) return;

    vector<vector<int> > depths(G[v].size());
    vector<int> all_depths;
    for (size_t i = 0; i < G[v].size(); ++i) {
        if (centroid[G[v][i].to]) continue;
        enumerate_depths(G[v][i].to, -1, centroid, G, depths[i]);
        sort(all(depths[i]));
        depths[i].erase(unique(all(depths[i])), depths[i].end());  // 部分木ごとに同じ深さの要素は1つだけ保存
        for (size_t j = 0; j < depths[i].size(); ++j) depths[i][j] += G[v][i].cost;
        all_depths.insert(all_depths.end(), all(depths[i]));
    }
    sort(all(all_depths));

    // 重心を端点とするパスを確認
    for (size_t t = 0; t < xs.size(); ++t) {
        if (found[t]) continue;
        vector<int>::iterator lb = lower_bound(all(all_depths), xs[t]);
        if (lb != all_depths.end() && *lb == xs[t]) {
            found[t] = true;
            ++found_count;
        }
    }

    // 重心を通り、重心以外の頂点を端点とするパスを確認
    for (size_t t = 0; t < xs.size(); ++t) {
        if (found[t]) continue;
        for (size_t i = 0; i < G[v].size(); ++i) {
            if (centroid[G[v][i].to]) continue;
            for (size_t j = 0; j < depths[i].size(); ++j) {
                int d = depths[i][j];
                vector<int>::iterator lb = lower_bound(all(all_depths), xs[t] - d);
                if (lb != all_depths.end() && *lb == xs[t] - d) {
                    vector<int>::iterator lb2 = lower_bound(all(depths[i]), xs[t] - d);
                    // 同じ部分木に深さxs[t]-dの頂点が存在する場合はイテレーターの次の要素を見る
                    if (lb2 != depths[i].end() && *lb2 == xs[t] - d) {
                        lb++;
                        if (lb != all_depths.end() && *lb == xs[t] - d) {
                            found[t] = true;
                            ++found_count;
                            break;
                        }
                    } else {
                        found[t] = true;
                        ++found_count;
                        break;
                    }
                }
            }
            if (found[t]) break;
        }
    }
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int n;
        cin >> n;
        if (n == 0) break;
        cnt++;
        vector<vector<edge> > G(n);

        rep(i, n) {
            while (true) {
                int to;
                cin >> to;
                if (to == 0) break;
                ll cost;
                cin >> cost;
                to--;
                G[i].push_back(edge(to, cost));
                G[to].push_back(edge(i, cost));
            }
        }
        vector<int> x;
        while (true) {
            int xi;
            cin >> xi;
            if (xi == 0) break;
            x.push_back(xi);
        }
        xs = x;
        sort(xs.begin(), xs.end());
        xs.erase(unique(xs.begin(), xs.end()), xs.end());

        found = vector<bool>(xs.size());
        found_count = 0;
        CentroidDecomposition<edge, solve> cd(G);

        cd.solve();
        rep(i, x.size()) {
            int idx = lower_bound(all(xs), x[i]) - xs.begin();
            cout << (found[idx] ? "AYE" : "NAY") << "\n";
        }
        cout << ".\n";
    }
}
```