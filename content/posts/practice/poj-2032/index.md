+++
title = "POJ 2032 Square Carpets(AOJ 1128,BOJ 22814)"
date = 2025-03-04T13:53:30+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2032
https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1128
https://www.acmicpc.net/problem/22814

https://vjudge.net/problem/POJ-2032
https://vjudge.net/problem/Aizu-1128
https://vjudge.net/problem/Baekjoon-22814
<!--more-->
## 問題概要
- $H\times W$の0,1からなるグリッドが与えられる
- 幾つかの正方形領域を重ねることで0の部分を覆わないようにすべての1の部分を覆う
- 必要な正方形領域の個数の最小値を求めよ
### 制約
- $H,W\leq 10$
## 解法メモ
- ある場所を左上に正方形を置く場合、validな範囲で最大のものよりも小さい正方形を置く意味はないので、常に最大としてよい

- 基本的にはグリッドの各マスを順にみていき、そこに正方形を置くかおかないかで分岐して探索していくだけだが、以下のような高速化をしたら通った
	- 正方形を置いても覆える領域が増えない（既に置いた正方形の範囲内にしか置けない）場合には置かない
	- 以降、今置くことと同等以上の選択肢が存在するなら後で配置する
		- ↓の「※ 例」を参照
		- 4割くらい実行時間削れたが、TLが緩いのであまり約に立たなかった
	- mapでメモ化する
	- 制約が小さいので状態を表すのにbit演算を使う

```
※ 例
(0,0)に2x2の正方形が配置されている場合、
##....
##....

次に(0,1)に置いて↓のように2マス増やすよりも
###...
###...

(0,2)に置いたほうが効率がいい
####..
####..
```
## 実装例
<details>
<summary>沼ったところ</summary>

C++14まではオーバーロードされた演算子の部分式の評価順序が不定だったので、以下でmemo[p]が先に評価されてメモ化再帰ができなかった。
AOJはC++17なのでACできるがPOJはC++98なので全ケースで0を出力してWAになっていた
```cpp
int solve() {
	if (memo.count(p)) return memo[p];
	return memo[p] = solve(...);
}
```
参考 [厳密な式の評価順 [P0145R3]](https://cpprefjp.github.io/lang/cpp17/expression_evaluation_order.html)
</details>




最大正方形はライブラリ化しているので[AOJ DPL_3_A Largest Square](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DPL_3_A&lang=ja)を貼った

int128を使ったver.
POJのコンパイラにはint128がないのでCEする
```cpp
#include <algorithm>
#include <iostream>
#include <map>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
#define inf int(2e9)
using namespace std;

typedef __int128_t lllint;

// AOJ DPL3A Largest Square
// dp[i][j] = (i,j)を左上として配置できる最大の正方形のサイズ
vector<vector<int> > largest_square(const vector<vector<bool> > &v) {
    if (v.empty() || v[0].empty()) return vector<vector<int> >(v.size());
    const int h = v.size(), w = v[0].size();
    vector<vector<int> > dp(h, vector<int>(w, 0));
    for (int i = h - 1; i >= 0; --i) {
        for (int j = w - 1; j >= 0; --j) {
            if (v.at(i).at(j)) {
                if (i == h - 1 || j == w - 1) {
                    dp[i][j] = 1;
                } else {
                    dp[i][j] = min(dp[i + 1][j + 1], min(dp[i + 1][j], dp[i][j + 1])) + 1;
                }
            }
        }
    }
    return dp;
}

int h, w;
vector<vector<lllint> > square;  // 各場所を左上として配置する時の最大サイズの正方形のビット表現
lllint dst;                      // 最終的な配置状態

map<pair<lllint, int>, int> memo;

int solve(lllint board = 0, int idx = 0, int cnt = 0) {
    if (board == dst) return cnt;  // 全ての正方形が配置できた時点で配置した正方形の数をreturn

    pair<lllint, int> p = make_pair(board, cnt);
    if (memo.count(p)) return memo[p];

    int i = idx / w, j = idx % w;

    if (!(dst >> idx & 1)) {  // (i,j)が正方形を配置してはいけないマスの場合
        return memo[p] = solve(board, idx + 1, cnt);
    }

    if (!(board >> idx & 1)) {  // まだ(i,j)に正方形を配置していない場合は今後置けないので必ず置く
        return memo[p] = solve(board | square[i][j], idx + 1, cnt + 1);
    }

    int res = solve(board, idx + 1, cnt);
    lllint new_region = (~board) & square[i][j];  // 正方形を置いて新しく覆える領域
    if (new_region) {                             // 置いて覆える領域が広がる場合のみ置く
        bool ok = true;
        rep2(k, j + 1, w) {  // 以降、今置くこととの同等以上の選択肢が存在するならそちらに任せる
            if (!(dst >> (i * w + k) & 1)) continue;
            lllint new_region2 = (~board) & square[i][k];
            if ((new_region & new_region2) == new_region) {
                ok = false;
                break;
            }
        }
        if (ok) {
            res = min(res, solve(board | square[i][j], idx + 1, cnt + 1));
        }
    }
    return memo[p] = res;
}

int main() {
    while (true) {
        cin >> w >> h;
        if (h == 0 && w == 0) break;
        dst = 0;
        vector<vector<bool> > v(h, vector<bool>(w));
        rep(i, h) rep(j, w) {
            int x;
            cin >> x;
            v[i][j] = x == 1;
            if (v[i][j]) dst |= lllint(1) << (i * w + j);
        }

        // 各場所を左上として正方形を配置する時、配置できる最大サイズの正方形のビット表現を求める
        vector<vector<int> > square_sizes = largest_square(v);
        square = vector<vector<lllint> >(h, vector<lllint>(w));
        rep(i, h) rep(j, w) {
            if (v[i][j]) {
                lllint b = 0;
                rep2(k, i, i + square_sizes[i][j]) {
                    rep2(l, j, j + square_sizes[i][j]) { b |= lllint(1) << (k * w + l); }
                }
                square[i][j] = b;
            }
        }

        memo.clear();
        cout << solve() << endl;
    }
}
```

最低限の機能を入れたint128を自作したver.

```cpp
#include <algorithm>
#include <iostream>
#include <map>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
#define inf int(2e9)
using namespace std;

struct Uint128 {
    typedef unsigned long long ui;
    ui bits[2];
    Uint128(const ui& val = 0ull) { bits[0] = val, bits[1] = 0; }
    Uint128(const Uint128& l) { bits[0] = l.bits[0], bits[1] = l.bits[1]; }

    bool operator[](int i) const { return bits[i / 64] >> (i % 64) & 1; }
    int popcount(ui x) const {
        int res = 0;
        while (x) {
            if (x & 1) res++;
            x >>= 1;
        }
        return res;
    }
    void set(int i, bool val = true) {
        if (val) bits[i / 64] |= 1ull << (i % 64);
        else bits[i / 64] &= ~(1ull << (i % 64));
    }
    Uint128 operator~() const {
        Uint128 res;
        res.bits[0] = ~bits[0];
        res.bits[1] = ~bits[1];
        return res;
    }
    Uint128& operator&=(const Uint128& rhs) {
        bits[0] &= rhs.bits[0];
        bits[1] &= rhs.bits[1];
        return *this;
    }
    Uint128& operator|=(const Uint128& rhs) {
        bits[0] |= rhs.bits[0];
        bits[1] |= rhs.bits[1];
        return *this;
    }
    Uint128 operator&(const Uint128& rhs) const { return Uint128(*this) &= rhs; }
    Uint128 operator|(const Uint128& rhs) const { return Uint128(*this) |= rhs; }
    int count() const { return popcount(bits[0]) + popcount(bits[1]); }
    bool operator<(const Uint128& rhs) const {
        if (bits[1] != rhs.bits[1]) return bits[1] < rhs.bits[1];
        return bits[0] < rhs.bits[0];
    }
    bool operator==(const Uint128& rhs) const { return bits[0] == rhs.bits[0] && bits[1] == rhs.bits[1]; }
};

// AOJ DPL3A Largest Square
// dp[i][j] = (i,j)を左上として配置できる最大の正方形のサイズ
vector<vector<int> > largest_square(const vector<vector<bool> >& v) {
    if (v.empty() || v[0].empty()) return vector<vector<int> >(v.size());
    const int h = v.size(), w = v[0].size();
    vector<vector<int> > dp(h, vector<int>(w, 0));
    for (int i = h - 1; i >= 0; --i) {
        for (int j = w - 1; j >= 0; --j) {
            if (v.at(i).at(j)) {
                if (i == h - 1 || j == w - 1) {
                    dp[i][j] = 1;
                } else {
                    dp[i][j] = min(dp[i + 1][j + 1], min(dp[i + 1][j], dp[i][j + 1])) + 1;
                }
            }
        }
    }
    return dp;
}

int h, w;
vector<vector<Uint128> > square;  // 各場所を左上として配置する時の最大サイズの正方形のビット表現
Uint128 dst;                      // 最終的な配置状態

map<pair<Uint128, int>, int> memo;

int solve(Uint128 board, int idx = 0, int cnt = 0) {
    if (board == dst) return cnt;  // 全ての正方形が配置できた時点でreturn

    pair<Uint128, int> p = make_pair(board, cnt);
    if (memo.count(p)) return memo[p];

    int i = idx / w, j = idx % w;

    if (dst[idx] == 0) {  // (i,j)が正方形を配置してはいけないマスの場合
        int res = solve(board, idx + 1, cnt);
        return memo[p] = res;
    }
    if (board[idx] == 0) {  // まだ(i,j)に正方形を配置していない場合は今後置けないので必ず置く
        int res = solve(board | square[i][j], idx + 1, cnt + 1);
        return memo[p] = res;
    }

    int res = solve(board, idx + 1, cnt);
    Uint128 new_region = (~board) & square[i][j];  // 正方形を置いて新しく覆える領域
    if (new_region.count() > 0) {                  // 置いて覆える領域が広がる場合のみ置く
        bool ok = true;
        rep2(k, j + 1, w) {  // 以降、今置くこととの同等以上の選択肢が存在するならそちらに任せる
            if (dst[i * w + k] == 0) continue;
            Uint128 new_region2 = (~board) & square[i][k];
            if ((new_region & new_region2) == new_region) {
                ok = false;
                break;
            }
        }
        if (ok) {
            res = min(res, solve(board | square[i][j], idx + 1, cnt + 1));
        }
    }
    return memo[p] = res;
}

int main() {
    while (true) {
        cin >> w >> h;
        if (h == 0 && w == 0) break;
        dst = 0;
        vector<vector<bool> > v(h, vector<bool>(w));
        rep(i, h) rep(j, w) {
            int x;
            cin >> x;
            v[i][j] = x == 1;
            if (v[i][j]) {
                dst.set(i * w + j, true);
            }
        }

        // 各場所を左上として正方形を配置する時、配置できる最大サイズの正方形のビット表現を求める
        vector<vector<int> > square_sizes = largest_square(v);
        square = vector<vector<Uint128> >(h, vector<Uint128>(w));
        rep(i, h) rep(j, w) {
            if (v[i][j]) {
                Uint128 b;
                rep2(k, i, i + square_sizes[i][j]) {
                    rep2(l, j, j + square_sizes[i][j]) { b.set(k * w + l, true); }
                }
                square[i][j] = b;
            }
        }

        memo.clear();
        cout << solve(Uint128(0)) << endl;
    }
}
```
