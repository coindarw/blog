+++
title = "POJ 2115 C Looooops(BOJ 6670, Gym 100811C)"
date = 2025-03-04T13:53:38+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2115
https://www.acmicpc.net/problem/6670
https://codeforces.com/gym/100811/attachments

https://vjudge.net/problem/POJ-2115
https://vjudge.net/problem/Baekjoon-6670
https://vjudge.net/problem/Gym-100811C
<!--more-->
## 問題概要
```c
for (variable = A; variable != B; variable += C)
  statement;
```
- 上のような、C形式のfor文があったとする。各変数が$k$ビット符号なし整数型（$\bmod {2^k}$で計算される）だとする．
- $A,B,C,k$が与えられるので、statementは何回実行されるか答えよ。
- 実行が終わらない場合は`FOREVER`と出力
### 制約
- $1\leq k\leq 32$
- $0\leq A,B,C\leq 2^k$
- 0,0,0,0で入力終了を表すマルチケース

## 解法メモ
- $A+Ci\equiv B\pmod {2^k}$を満たす最小の非負整数$i$を見つけたい。


- $Ci\equiv B-A\pmod {2^k}$を考える。Cの逆元が求められればうまくいきそう
- $(B-A)\%2^k$が$\gcd(C, 2^k)$で割り切れる時この解は存在する

- gcdで両辺を割った式で計算してから最後に元に戻せばよい。
## 実装例
```cpp
#include <algorithm>
#include <cstdio>
typedef long long ll;
using namespace std;

ll mod_inv(ll a, ll M) {
    ll b = M, u = 1, v = 0;
    while (b) {
        ll t = a / b;
        a -= t * b;
        swap(a, b);
        u -= t * v;
        swap(u, v);
    }
    u %= M;
    if (u < 0) u += M;
    return u;
}

ll gcd(ll a, ll b) {
    if (a < b) swap(a, b);
    if (b == 0) return a;
    return gcd(b, a % b);
}

int main() {
    while (true) {
        ll a, b, c, k;
        scanf("%lld%lld%lld%lld", &a, &b, &c, &k);
        if (a == 0 && b == 0 && c == 0 && k == 0) break;
        ll mod = 1LL << k;
        ll g = gcd(c, mod);
        ll d = (b - a + mod) % mod;
        if (d % g == 0) {
            ll ans = mod_inv(c / g, mod / g) * (d / g);
            ans %= mod / g;
            printf("%lld\n", ans);
        } else {
            printf("FOREVER\n");
        }
    }
}
```