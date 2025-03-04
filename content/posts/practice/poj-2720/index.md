+++
title = "POJ 2720 Last Digits"
date = 2025-03-04T13:53:46+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2720

https://vjudge.net/problem/POJ-2720
<!--more-->
## 問題概要
- 整数 $b,i$ が与えられたとき、関数 $f(x)$ を以下のように再帰的に定義する
- $f(x) = b^{f(x-1)} (x>0)$
- $f(0)=1$

- $f(i)$ の最後の $n$ 桁を求めよ（$n$桁未満の場合0埋め）
### 入力
- マルチテストケース
- $1 \leq b \leq 100$
- $1 \leq i \leq 100$
- $1 \leq n \leq 7$
-  $b = 0$ の時入力終了
## 解法メモ
※POJではコンパイラのバージョンが古すぎてCEになるので提出できないが 、 https://trap.jp/post/1444/ を貼ると解ける。テンプレート再帰でとても簡潔に書けていてすごい（語彙力）


- $\mod {10^7}$で計算すれば$n$が7未満の場合も割ればいいので、$n=7$に固定して考える。

- とりあえず$10^7$と$b$が互いに素だとすると、
- $f(i-1)\pmod {\phi(10^7)}$の値を使えば$\mod 10^7$の値が計算できる。その値の計算のために$f(i-2)\pmod{\phi(\phi(10^7))}$を計算して...というように再帰していけばよい。

- 実際には、$b$と$10^7$が互いに素でない場合もあるので、mod_pow関数を少し改造し、累乗の結果が法$M$以上であれば$(x^y\mod M)+M$を返すようにしておけば簡単に実装できる。

- POJのTLが厳しいためメモ化などいろいろ高速化が必要
## 実装例
cin/cout版
```cpp
#include <cstring>
#include <iomanip>
#include <iostream>
#include <map>
typedef long long ll;
using namespace std;

const int M = 10000000;  // 10^7
map<int, int> memo;
int totient(int n) {
    if (memo.find(n) != memo.end()) return memo[n];
    int res = n;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            res -= res / i;
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) res -= res / n;
    return memo[n] = res;
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

int memo2[21][101][101];

int solve_sub(int b, int i, int mod, int cnt = 0) {  // (b^f(i) < mod) ? b^f(i) : (b^f(i) % mod + mod)
    if (mod == 1) return 1;                          // b>0 -> b^f(i) % 1 + 1
    if (i == 0) return 1;
    if (memo2[cnt][b][i] != -1) return memo2[cnt][b][i];
    int phi = euler_phi(mod);
    int prv = solve_sub(b, i - 1, phi, cnt + 1);
    ll res = mod_pow_customized(b, prv, mod);
    return memo2[cnt][b][i] = mod_pow_customized(b, prv, mod);
}
int solve(int b, int i, int n) {
    int mod = 1;
    while (n--) mod *= 10;
    return solve_sub(b, i, M) % mod;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    memset(memo2, -1, sizeof(memo2));  // -1でメモ用配列を初期化
    while (true) {
        int b;
        cin >> b;
        if (b == 0) break;
        int i, n;
        cin >> i >> n;
        cout << setfill('0') << right << setw(n) << solve(b, i, n) << '\n';
    }
}
```

scanf/printf版
```cpp
#include <cstdio>
#include <cstring>
#include <map>
typedef long long ll;
using namespace std;

const int M = 10000000;  // 10^7
map<int, int> memo;
int euler_phi(int n) {
    if (memo.find(n) != memo.end()) return memo[n];
    int res = n;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            res -= res / i;
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) res -= res / n;
    return memo[n] = res;
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

int memo2[21][101][101];

int solve_sub(int b, int i, int mod, int cnt = 0) {  // (b^f(i) < mod) ? b^f(i) : (b^f(i) % mod + mod)
    if (mod == 1) return 1;                          // b>0 -> b^f(i) % 1 + 1
    if (i == 0) return 1;
    if (memo2[cnt][b][i] != -1) return memo2[cnt][b][i];
    int phi = euler_phi(mod);
    int prv = solve_sub(b, i - 1, phi, cnt + 1);
    ll res = mod_pow_customized(b, prv, mod);
    return memo2[cnt][b][i] = mod_pow_customized(b, prv, mod);
}
int solve(int b, int i, int n) {
    int mod = 1;
    while (n--) mod *= 10;
    return solve_sub(b, i, M) % mod;
}

int main() {
    memset(memo2, -1, sizeof(memo2));  // -1でメモ用配列を初期化
    while (true) {
        int b;
        scanf("%d", &b);
        if (b == 0) break;
        int i, n;
        scanf("%d%d", &i, &n);
        char format[20];
        sprintf(format, "%%0%dd\n", n);  // n桁0埋めのフォーマット文字列
        printf(format, solve(b, i, n));
    }
}
```