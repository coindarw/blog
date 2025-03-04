+++
title = "POJ 1284 Primitive Roots"
date = 2025-03-04T13:53:20+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1284

https://vjudge.net/problem/POJ-1284

<!--more-->
## 問題概要
- 奇素数$p$が与えられたとき、整数$x(0<x<p)$は、集合$\{(x^i\bmod p)\mid 1\leq i\leq p-1\}$と$\{1,\dots,p-1\}$が等しいとき原始根と呼ばれる。
- 奇素数$3\leq p<65536$が与えられるので原始根の数を出力せよ
- マルチケースでケース数は与えられない

## 解法メモ
- 原始根に関する基本的な知識が聞かれている感じの問題。
- $p-1$以下で$p-1$と互いに素な整数の個数が答えになるので、オイラーのφ関数を貼ればOK。
## 実装例
```cpp
#include <cstdio>

int euler_phi(int n) {
    int res = n;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            res -= res / i;
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) res -= res / n;
    return res;
}

int main() {
    int p;
    while (scanf("%d", &p) != EOF) {
        printf("%d\n", euler_phi(p - 1));
    }
}
```