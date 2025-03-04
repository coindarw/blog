+++
title = "POJ 2749 Building roads"
date = 2025-03-04T13:53:50+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++


http://poj.org/problem?id=2749
https://acm.hdu.edu.cn/showproblem.php?pid=1815

https://vjudge.net/problem/POJ-2749
https://vjudge.net/problem/HDU-1815
POJだとTLがきついのでHDUでAC。**HDUでは問題文には書かれていないがマルチケースになっているので注意**
<!--more-->
## 問題概要
- 2次元平面状にN個の納屋があり、それぞれの位置は$(x_i,y_i)$である。各納屋には牛が何頭かいる
- 牛たちが動き回れるように、これらの納屋を結ぶ道路を建設したい


- 転送ポイント$S_1,S_2$を用意し、$S_1,S_2$の間に道路を建設したうえで、各納屋とどちらかの転送ポイントの間に道路を建設する。$S_1, S_2$の座標は$(x_{S_1}, y_{S_1}),(x_{S_2}, y_{S_2})$


- $A$個の納屋のペア$(a_i, b_i)$は同じ転送ポイントに接続する必要がある
- $B$個の納屋のペア$(c_i,d_i)$は異なる転送ポイントに接続する必要がある
- 上の$A+B$個の条件を満たしたうえで、各納屋のペアの間を移動する際のマンハッタン距離の最大値を最小化したい。
	- 例えば$(x_i,y_i)$が$S_1$に、$(x_j,y_j)$が$S_3$に接続しているなら距離は以下の値
	$$|x_{S_1}-x_i|+|y_{S_1}-y_i|+|x_{S_2}-x_{S_1}|+|y_{S_2}-y_{S_1}|+|x_j-x_{S_2}|+|y_j-y_{S_2}|$$


- そのような建設計画が存在するか判定、存在するなら各ペア間の距離の最大値を出力。
### 制約
- $2\leq N\leq 500$
- $0\leq A\leq 1000$
- $0\leq B\leq 1000$

- $-10^6\leq x_i,y_i,x_{S_1},y_{S_1},x_{S_2},y_{S_2}\leq 10^6$
## 解法メモ
- 最大値の最小化なので二分探索が浮かぶ。
- 納屋$i$の接続先をbool値$s_i$で表し、$S_1$に接続しているならfalse, $S_2$に接続しているならtrueとして、2-SATを使うと解くことができる


- 「納屋の各ペア$i,j$について$A+B$個の条件を満たし、かつ$i$から$j$に移動する際の距離が$k$以下になるような割り当てが存在するかどうか」を判定すればよい。


- $A$個の「$s_i=s_j$」条件
	- xorが0になればいいので、$(\lnot s_i \lor s_j) \land (s_i \lor\lnot s_j)=1$
- $B$個の「$s_i\neq s_j$」条件
	- xorが1になればいいので、$(s_i \lor s_j)\land(\lnot s_i \lor \lnot s_j)=1$


- 距離については、納屋の各ペア$i,j$についての、$i,j$の接続先の組み合わせ4通り
- (S_1,S_1),(S_1,S_2),(S_2,S_1),(S_2,S_2)$ それぞれについて距離が$k$を超える組み合わせが常にFalseになるように2-SATを使えばよい。

- 例
	- $i$が$S_1$に、$j$が$S_2$につながっていると距離$k$を超えてしまう場合、満たすべき条件は$\lnot(\lnot s_i\land s_j)=(s_i\lor \lnot s_j)$となる。
## 実装例
```cpp
#include <algorithm>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <iostream>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define rrep(i, n) for (int i = ((int)(n)-1); i >= 0; --i)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
#define inf int(1e9)
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

typedef pair<int, int> P;
int n, a, b;
P s1, s2;
vector<P> barns;
vector<P> condA, condB;

int manhattan(P p1, P p2) { return abs(p1.first - p2.first) + abs(p1.second - p2.second); }
int distance(int b1, bool c1, int b2, bool c2) {
    P p1 = barns[b1];
    P m1 = c1 ? s1 : s2;
    P m2 = c2 ? s1 : s2;
    P p2 = barns[b2];
    return manhattan(p1, m1) + manhattan(m1, m2) + manhattan(m2, p2);
}

int d[500][500][2][2];

twosat ts0;
bool cmp(int x) {
    twosat ts = ts0;  // 前計算しておいたものをコピー
    // rep(ai, a) {
    //     int i = condA[ai].first, j = condA[ai].second;
    //     ts.add_clause(i, 1, j, 1);
    //     ts.add_clause(i, 0, j, 0);
    // }
    // rep(bi, b) {
    //     int i = condB[bi].first, j = condB[bi].second;
    //     ts.add_clause(i, 1, j, 0);
    //     ts.add_clause(i, 0, j, 1);
    // }
    rep(i, n) rep2(j, i + 1, n) {
        rep(ti, 2) rep(tj, 2) {
            if (d[i][j][ti][tj] > x) {
                ts.add_clause(i, !ti, j, !tj);
            }
        }
    }
    return ts.satisfiable();
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (cin >> n >> a >> b) {
        cin >> s1.first >> s1.second >> s2.first >> s2.second;

        barns = vector<P>(n);
        rep(i, n) cin >> barns[i].first >> barns[i].second;

        condA = vector<P>(a);
        condB = vector<P>(b);
        rep(i, a) cin >> condA[i].first >> condA[i].second;
        rep(i, b) cin >> condB[i].first >> condB[i].second;
        rep(i, a) condA[i].first--, condA[i].second--;
        rep(i, b) condB[i].first--, condB[i].second--;

        // 前計算
        ts0 = twosat(n);
        rep(ai, a) {
            int i = condA[ai].first, j = condA[ai].second;
            ts0.add_clause(i, 1, j, 1);
            ts0.add_clause(i, 0, j, 0);
        }
        rep(bi, b) {
            int i = condB[bi].first, j = condB[bi].second;
            ts0.add_clause(i, 1, j, 0);
            ts0.add_clause(i, 0, j, 1);
        }
        rep(i, n) rep2(j, i + 1, n) {
            rep(ti, 2) rep(tj, 2) { d[i][j][ti][tj] = distance(i, ti, j, tj); }
        }

        if (!cmp(inf)) {
            cout << -1 << "\n";
        } else {
            int ok = inf, ng = 0;
            while (abs(ng - ok) > 1) {
                int mid = (ng + ok) / 2;
                if (cmp(mid)) ok = mid;
                else ng = mid;
            }
            if (ok == inf) ok = -1;
            cout << ok << "\n";
        }
    }
}
```