+++
title = "POJ 3688 Cheat in the Game"
date = 2025-03-04T13:54:15+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3688

https://vjudge.net/problem/POJ-3688

問題文が難読
<!--more-->

## 問題概要
- AliceとBobが以下のゲームを行う。石の個数$w$を1以上$M$以下の範囲で変えた時に、Aliceが必勝になる$w$の個数を求めよ
	- $N$枚のカードがあり、$i$枚目のカードには番号$A_i$が書かれている。これを使ってAlice, Bobゲームを行う。
	- はじめ石は$w$個あり、Alice, Bobは交互に以下の行動を行う
		- ランダムにカードを1枚引き、書かれている番号の数だけ山から石を除く。番号の数の方が大きい場合は引き直す
### 制約
- $1\leq N\leq 10^4$
- $1\leq M\leq 10^5$

TL : 6 sec
## 解法メモ
- $w$をある奇数枚のカードの合計として表すことができ、かつ偶数枚のカードの合計として表すことができない場合、Aliceは必勝。それ以外の場合、Bobが勝つ可能性が存在する。

- DPすると計算量$O(NM)$だが、TLが長いのでギリギリ通った。

- 最初bitset高速化が想定解法かと思ったが、これだとTL6秒も取らない気がするので、「$O(NM)$だけどTLが長いので、配列再利用とin-place更新ができれば通ります」という趣旨の問題なのではないかと思う
	- [POJ Founder Monthly Contest – 2008.08.31](http://poj.org/showcontest?contest_id=1322) Editorialなどはなさそう（？）なので実際の想定解はわからず
## 実装例
bitset高速化解 GCCで出すと375ms
```cpp
#include <stdio.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define rrep(i, n) for (int i = ((int)(n)-1); i >= 0; --i)

#define B 64
#define M 100009
#define K ((M + 1 + 63) / B)
unsigned long long dp[2][K];
int a[10009];
int main() {
    while (1) {
        int n, m;
        scanf("%d%d", &n, &m);
        if (n == 0 && m == 0) break;
        rep(i, n) scanf("%d", &a[i]);

        rep(i, (m + 1 + 63) / B) dp[0][i] = dp[1][i] = 0;
        dp[0][0] = 1;
        rep(i, n) {
            rrep(j, (m + 1 + 63) / B) {
                int plus_idx = j + a[i] / B;
                if (a[i] % B != 0 && plus_idx + 1 < K) {
                    dp[1][plus_idx + 1] |= dp[0][j] >> (B - (a[i] % B));
                    dp[0][plus_idx + 1] |= dp[1][j] >> (B - (a[i] % B));
                }
                if (plus_idx < K) {
                    unsigned long long tmp = dp[1][j];
                    dp[1][plus_idx] |= dp[0][j] << (a[i] % B);
                    dp[0][plus_idx] |= tmp << (a[i] % B);
                }
            }
        }
        int ans = 0;
        reps(i, m) {
            int f0 = (dp[0][i / B] >> (i % B)) & 1;
            int f1 = (dp[1][i / B] >> (i % B)) & 1;
            if (!f0 && f1) {
                ans++;
            }
        }
        printf("%d\n", ans);
    }
}
```

普通のDP解。
LanguageをG++にすると通ったが、C++だと通らなかった。実装が良くない気がする
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define rrep(i, n) for (int i = ((int)(n)-1); i >= 0; --i)
using namespace std;

#define M 100009
bool dp[2][M];
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int n, m;
        cin >> n >> m;
        if (n == 0 && m == 0) break;
        vector<int> a(n);
        rep(i, n) cin >> a[i];

        rep(i, 2) rep(j, m + 1) dp[i][j] = false;
        dp[0][0] = true;
        rep(i, n) {
            rrep(j, m + 1) {
                if (j + a[i] <= m) {
                    dp[1][j + a[i]] |= dp[0][j];
                    dp[0][j + a[i]] |= dp[1][j];
                }
            }
        }
        int ans = 0;
        reps(i, m) {
            if (!dp[0][i] && dp[1][i]) {
                ans++;
            }
        }
        cout << ans << '\n';
    }
}
```