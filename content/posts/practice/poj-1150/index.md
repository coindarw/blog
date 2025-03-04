+++
title = "POJ 1150 The Last Non-zero Digit(UVa 10212)"
date = 2025-03-04T13:53:16+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1150
https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=1153

https://vjudge.net/problem/POJ-1150
https://vjudge.net/problem/UVA-10212
<!--more-->
## 問題概要
- 2つの整数$N,M$が与えられます。${}_NP_M$を10進表記した時、0でない最小の桁の値を求めよ
	- （問題文だと${}^NP_M$という見たことのない表記だが順列$\dfrac{N!}{(N-M)!}$のこと）

### 制約
- マルチケースでケース数は与えられない
- $0\leq N\leq2\times10^7, 0\leq M\leq N$
### 例
- $N=25,M=6$の時${}_NP_M=\dfrac{25!}{(25-6)!}=127512000$なので2を出力する

## 解法メモ
- $\dfrac{(N!を10でできる限り割ったもの)}{(N-M)!}$を求めればよさそう。
- 階乗のmod逆元をmod 10で求めたくなるが、$0!,1!$を除いて偶数なので10と互いに素ではなく、この方法が使えない。
- そこで、階乗の値をあらかじめ2,5でできる限り割っておき、あとで必要な分をかければ良い。

- n!に2,5がいくつ含まれるかは、ルジャンドルの定理を使うことで簡単に求めることができる。（大学入試でよくある100!は5で何回割れる？というやつ）

- MLが非常に厳しいため、階乗の値を保存しようと思うとchar型でないと間に合わない
## 実装例
```cpp
#include <algorithm>
#include <cstdio>
#include <iostream>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
using namespace std;

char factorial_wo25[20000000 + 1];  // factorial_wo25[i] = i!を2,5で割れる限り割ったもの mod 10

const int M = 10;

int mod_inv(int a) {  // aがMと互いに素、つまり2,5以外なら逆元を求められる
    int b = M, u = 1, v = 0;
    while (b) {
        int t = a / b;
        a -= t * b;
        swap(a, b);
        u -= t * v;
        swap(u, v);
    }
    u %= M;
    if (u < 0) u += M;
    return u;
}
int mod_pow(int x, int n) {
    int res = 1;
    while (n > 0) {
        if (n & 1) res *= x, res %= M;
        x *= x, x %= M;
        n >>= 1;
    }
    return res;
}

int factor_in_factorial(int n, int p) {  // n!にpが何個含まれるか
    int res = 0;
    int q = p;
    while (n >= q) {
        res += n / q;
        q *= p;
    }
    return res;
}

int inv[10];  // mod 10での逆元

int main() {
    factorial_wo25[0] = 1;
    reps(i, 20000000) {
        int x = i;
        while (x % 2 == 0) x /= 2;
        while (x % 5 == 0) x /= 5;
        factorial_wo25[i] = (factorial_wo25[i - 1] * x) % 10;
    }
    rep(i, 10) {
        if (i != 0 && i != 2 && i != 5) {
            inv[i] = mod_inv(i);
        }
    }
    int n, m;
    while (scanf("%d%d", &n, &m) != EOF) {
        int c2 = factor_in_factorial(n, 2) - factor_in_factorial(n - m, 2);
        int c5 = factor_in_factorial(n, 5) - factor_in_factorial(n - m, 5);
        int min_c2c5 = min(c2, c5);
        int ans = (factorial_wo25[n] * inv[factorial_wo25[n - m]] * mod_pow(2, c2 - min_c2c5) * mod_pow(5, c5 - min_c2c5)) % 10;
        cout << ans << '\n';
    }
}
```