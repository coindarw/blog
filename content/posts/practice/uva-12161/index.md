+++
title = "UVa 12161 Ironman Race in Treeland"
date = 2025-03-04T13:54:29+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=3313

https://vjudge.net/problem/UVA-12161
<!--more-->
## 問題概要
- $N$頂点の木が与えられる
- 各辺にはその辺を選ぶことによる損失$D_i$と距離$L_i$が設定されている
- 以下の条件を満たすように2点を選ぶ時の距離$L_i$の合計の最大値を求めよ
	- パス上の$D_i$の合計が$M$を超えない
### 制約
- $1\leq T\leq 10$
- $1\leq N\leq 3\times 10^4$
- $1\leq M\leq 10^8$
- $1\leq D_i,L_i\leq 10^3$
## 解法メモ
- 重心分解すると根を通るパスだけ考えればよくなる。
- パスの片方の頂点$s$を固定した時に、根を通り、損失の合計が$M$を超えないパスのうち距離が最長になるものを高速に求めたい
- （根からのパスの損失の合計, 根からのパスの距離）のリストを作ってから損失でソートし、距離をSegTreeに入れておけば二分探索＋RMQで処理できる
- 自分の方向に向かう頂点を数えてしまわないように、適宜値を$-\infty$等で書き換える必要がある

## 実装例
```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <functional>
#include <iostream>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
using namespace std;
#define all(v) v.begin(), v.end()

template <typename EDGE>
struct CentroidDecomposition {
    using F = std::function<void(int, const vector<bool> &, const vector<vector<EDGE>> &)>;
    const vector<vector<EDGE>> &G;
    vector<bool> centroid;
    vector<int> subtree_size;

    vector<int> tree_parent;
    vector<vector<int>> tree;
    F f;

    CentroidDecomposition(const vector<vector<EDGE>> &_G, F _f)
        : G(_G),
          centroid(_G.size()),
          subtree_size(_G.size()),
          tree_parent(_G.size(), -1),
          tree(_G.size()),
          f(_f) {}

    void add_edge(int u, int v) {
        G[u].push_back(v);
        G[v].push_back(u);
    }

    void solve() { solve(0, 1, -1); }

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
    void solve(int v, int d, int p) {
        compute_subtree_size(v, -1);
        int s = search_centroid(v, -1, subtree_size[v]).second;
        tree_parent[s] = p;
        if (p != -1) tree[p].push_back(s);
        centroid[s] = d;
        for (size_t i = 0; i < G[s].size(); ++i) {
            if (centroid[G[s][i].to]) continue;
            solve(G[s][i].to, d + 1, s);
        }
        f(s, centroid, G);
        centroid[s] = false;
    }
};

template <typename T, T (*op)(T, T), T (*e)()>
class SegTree {
   private:
    int n, _n;
    std::vector<T> dat;

    T _query(int a, int b, int p, int l, int r) {
        if (a >= r || b <= l) {
            return e();
        } else if (a <= l && r <= b) {
            return dat.at(p);
        }
        int mid = (l + r) / 2;
        return op(_query(a, b, p * 2 + 1, l, mid), _query(a, b, p * 2 + 2, mid, r));
    }

   public:
    SegTree(int n) : SegTree(std::vector<T>(n, e())) {}

    SegTree(const std::vector<T> &v) : _n(int(v.size())) {
        this->n = 1;
        while (this->n < _n) {
            this->n *= 2;
        }
        dat = std::vector<T>(2 * this->n - 1);
        for (int i = 0; i < _n; i++) {
            set(i, v.at(i));
        }
    }

    T query(int l, int r) {
        assert(0 <= l && l <= r && r <= _n);
        if (l == r) return e();
        return _query(l, r, 0, 0, n);
    }

    void set(int p, T a) {
        assert(0 <= p && p < _n);
        p += n - 1;
        dat.at(p) = a;
        while (p > 0) {
            p = (p - 1) / 2;
            dat.at(p) = op(dat.at(p * 2 + 1), dat.at(p * 2 + 2));
        }
    }

    T get(int p) {
        assert(0 <= p && p < _n);
        return dat.at(p + n - 1);
    }
};

int e() { return -inf; }
int op(int a, int b) { return max(a, b); }

int main() {
    int Q;
    cin >> Q;
    rep(__case, Q) {
        int n, m;
        cin >> n >> m;
        struct edge {
            int to;
            int damage, length;
            edge(int _to, int _damage, int _length) : to(_to), damage(_damage), length(_length) {}
        };
        vector<vector<edge>> G(n);
        rep(i, n - 1) {
            int u, v, d, l;
            cin >> u >> v >> d >> l;
            --u, --v;
            G[u].emplace_back(v, d, l);
            G[v].emplace_back(u, d, l);
        }

        int ans = 0;
        // 重心分解
        auto f = [&](int c, const vector<bool> &centroid, const vector<vector<edge>> &G) {
            vector<vector<tuple<int, int, int>>> damage_length(G[c].size());
            vector<tuple<int, int, int>> damage_length_all;

            int acc = 0;
            rep(i, G[c].size()) {
                // 重心の子方向にDFS、各方面にある(ダメージ, 距離, インデックス)を記録
                if (centroid[G[c][i].to]) continue;

                function<void(int, int, int, int)> dfs = [&](int v, int p, int dSum, int lSum) {
                    damage_length[i].emplace_back(dSum, lSum, acc);
                    damage_length_all.emplace_back(dSum, lSum, acc);
                    acc++;
                    for (const edge &e : G[v]) {
                        if (e.to == p) continue;
                        if (centroid[e.to]) continue;
                        dfs(e.to, v, dSum + e.damage, lSum + e.length);
                    }
                };
                dfs(G[c][i].to, c, G[c][i].damage, G[c][i].length);
            }
            sort(all(damage_length_all));
            vector<int> damage(damage_length_all.size()), length(damage_length_all.size());
            vector<int> revIdx(damage_length_all.size());  // ソート後の位置を逆引き
            rep(i, damage_length_all.size()) {
                damage[i] = get<0>(damage_length_all[i]);
                length[i] = get<1>(damage_length_all[i]);
                revIdx[get<2>(damage_length_all[i])] = i;
            }

            // 端点が根のパス
            rep(i, damage_length_all.size()) {
                if (damage[i] <= m) ans = max(ans, length[i]);
            }

            // 端点が根でないパス
            SegTree<int, op, e> st(length);
            rep(i, G[c].size()) {
                if (centroid[G[c][i].to]) continue;
                // 根まで行って戻るパスを数えないように子部分木の要素を-∞で埋めておく
                for (auto dli : damage_length[i]) {
                    int d, l, idx;
                    tie(d, l, idx) = dli;
                    st.set(revIdx[idx], e());
                }
                for (auto dli : damage_length[i]) {
                    int d, l, idx;
                    tie(d, l, idx) = dli;
                    int ub = upper_bound(all(damage), m - d) - damage.begin();
                    ans = max(ans, l + st.query(0, ub));
                }
                for (auto dli : damage_length[i]) {
                    int d, l, idx;
                    tie(d, l, idx) = dli;
                    st.set(revIdx[idx], l);
                }
            }
        };
        CentroidDecomposition<edge> cd(G, f);
        cd.solve();
        cout << "Case " << __case + 1 << " : " << ans << endl;
    }
}

```


