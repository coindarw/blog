+++
title = "GCJ Japan 2011 決勝B バクテリアの増殖(BOJ 12445, 12446)"
date = 2025-03-04T13:53:11+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://www.acmicpc.net/problem/12445 (Small)
https://www.acmicpc.net/problem/12446 (Large)

https://vjudge.net/problem/Baekjoon-12445
https://vjudge.net/problem/Baekjoon-12446
<!--more-->
## 問題概要
- ある時点で$x$個存在したとすると、1時間後には$x^x$個に増えるバクテリアがある。
- $A$個のバクテリアが$B$時間後に何個になっているか、$\mod C$で求めよ
### 制約
- テストケース数$1\leq T\leq 500$
- $1\leq A\leq 1000$
- $1\leq B\leq 1000$
- $1\leq C\leq 1000$

## 解法メモ
- [[POJ 2720 Last Digits]]同様オイラーの定理を使うと周期性が見えて解くことができる


- $a^n$は、$n<\phi(P)$であればそのまま計算すればよく、
- $n\geq P$なら$a^n\equiv a^{n\bmod\phi(P)+\phi(P)}\mod P$

- 今回の問題の場合、
	- 時刻t時点でのバクテリアの個数をmodで表したものを`f(t)`として、`g(mod, t) = (f(t) < mod) ? f(t)%mod : f(t)%mod + mod` を再帰的に求めていけばいい。
	- これは、`g(mod, t-1)^g(totient(mod), t-1)` と底はそのままのmodで、指数は$\mod \phi$で管理していけばよい。

## 実装例
```cpp
#include <bits/stdc++.h>
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
using ll = long long;
using namespace std;

ll totient(ll n) {
    ll res = n;
    for (ll i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            res -= res / i;
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) res -= res / n;
    return res;
}

ll mod_pow_customized(ll x, ll n, ll M) {  // (x^n < M) ? (x^n) : (x^n % M) + M
    if (M == 1) return bool(x || (x == 0 && n == 0));
    ll res = 1;
    bool f = false, f2 = false;
    while (n > 0) {
        if (n & 1) res *= x, f |= (res >= M || f2), res %= M;
        x *= x, f2 |= (x >= M), x %= M;
        n >>= 1;
    }
    return f ? (M + res) : res;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int t;
    cin >> t;
    reps(i, t) {
        ll a, b, c;
        cin >> a >> b >> c;
        map<pair<int, int>, int> memo;

        // 時刻t時点でのバクテリアの個数をf(t)として、g(mod, t) = (f(t) < mod) ? f(t)%mod : f(t)%mod + mod
        auto g = [&](auto g, ll mod, ll t) -> int {
            if (t == 0) return (a % mod) + (a >= mod ? mod : 0);
            if (memo.count({mod, t})) return memo[{mod, t}];
            int phi = totient(mod);
            int px = g(g, mod, t - 1);
            int py = g(g, phi, t - 1);
            return memo[{mod, t}] = mod_pow_customized(px, py, mod);
        };

        cout << "Case #" << i << ": " << (g(g, c, b) % c) << '\n';
    }
}
```

### 余談
https://en.wikipedia.org/wiki/Euler%27s_totient_function によれば、オイラーのφ関数、オイラーは記号に$\pi$を使っており、$\phi$を使ったのはガウス。totientという表現を使ったのはJames Joseph Sylvester。
オイラー自身は$\phi$ともtotientとも言っていないらしい