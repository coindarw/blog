+++
title = "POJ 1011 Sticks"
date = 2025-03-04T13:53:13+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1011

https://vjudge.net/problem/POJ-1011
<!--more-->
## 問題概要
- サイズ$N$の正整数からなる多重集合$A$が与えられる
- $A$をいくつかの多重集合$B_1, B_2, \cdots, B_k$に分割し、それらの合計が等しくなるようにする
- 合計$\sum B_1=\sum B_2=\cdots=\sum B_k$の最小値を求めよ
### 制約
- $1\leq A_i\leq 50$
- $1\leq N\leq 64$
## 解法メモ
- 合計としてあり得るものを小さい順に枝刈り全探索をするが、枝刈りのやり方にかなりいろいろあると思う

やったことを列挙する
- 合計を$\sum A_i$で割り切れない数、$\max(A_i)$未満の数にすることはできない
- 降順にソートする
	- 先に条件が厳しい側を選ぶことで、実装例コードの※のような枝刈りができる
- ある多重集合の最初の要素を決めるとき、現在選べる最初の要素を選んでうまくいかないならその後もうまくいかない
- dpで「以降の要素を使って合計$x$にできるか」をあらかじめ計算しておき、できないなら打ち切る
- 同じ値を区別しないようにする
	- （`a = {1, 1, 1}` の時に`a[1]+a[2], a[1]+a[3]` を同一視しない）


やったけどあまり効果がなかったもの
- dpで「min(以降の要素を使って合計$x$にできる方法の数, 64)」を数え上げておき、分割したい個数分なければ探索しない
	- ほぼ意味がないかdpにかかる時間分遅くなる
- 一定時間たってうまくいかなかったら打ち切り
	- テストケース数が多いうえ、適切に実装すれば数十msで終わるのでほぼ意味がなかった



## 実装例
```cpp
#include <algorithm>
#include <cstdio>
#include <ctime>
#include <numeric>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define rrep(i, n) for (int i = ((int)(n)-1); i >= 0; --i)
using namespace std;

vector<int> a;
vector<vector<bool> > dp;

typedef unsigned long long ull;
ull mask = 0xffffffffffffffff;
clock_t start;

bool rec(ull S, ull T, int last, int curSum, int sum, int l = 0) {
    if (S == (mask >> (64 - a.size()))) return true;  // (1ull << a.size()) - 1だとオーバーフローする
    if (!dp[last + 1][sum - curSum]) return false;    // 以降の要素を使ってもsumにできない

    int last_skipped = -1;     // 最後にスキップした要素と同じ要素は選ばない
    bool first = curSum == 0;  // これから選ぶ要素が最初の要素かどうか
    for (int j = last + 1; j < (int)a.size(); ++j) {
        if (S >> j & 1) continue;
        if (last_skipped == a[j]) continue;
        if (curSum + a[j] == sum) {
            if (rec(S | (1ull << j) | T, 0, -1, 0, sum, l + 1)) {
                return true;
            } else {  // 降順に見ているので、現在最大の要素でちょうどsumになるのにうまくいかないならその後もうまくいかない ※
                return false;
            }
        } else if (curSum + a[j] < sum) {
            if (rec(S, T | (1ull << j), j, curSum + a[j], sum, l)) {
                return true;
            } else {
                if (first) {  // 選んでいない中で最初の要素はどこかで必ず選ぶので、これを選んでうまくいかないならアウト
                    return false;
                }
                last_skipped = a[j];
            }
        }
    }
    return false;
}

int main() {
    while (true) {
        int n;
        scanf("%d", &n);
        if (n == 0) break;
        a.resize(n);
        rep(i, n) scanf("%d", &a[i]);

        sort(a.rbegin(), a.rend());  // 降順にソート

        int total = accumulate(a.begin(), a.end(), 0);
        int max_ = *max_element(a.begin(), a.end());

        // dp[i][j] := i番目以降の要素を使ってjを作れるか
        dp = vector<vector<bool> >(n + 1, vector<bool>(total + 1, false));
        dp[n][0] = true;
        rrep(i, n) rep(j, total + 1) {
            if (dp[i + 1][j]) {
                dp[i][j] = true;
                dp[i][j + a[i]] = true;
            }
        }

        for (int i = 1; i <= total; ++i) {
            if (total % i != 0) continue;
            if (i < max_) continue;
            if (!dp[0][i]) continue;
            if (i == total) {
                printf("%d\n", i);
                break;
            } else {
                if (rec(0, 0, -1, 0, i)) {
                    printf("%d\n", i);
                    break;
                }
            }
        }
    }
}

```