+++
title = "POJ 2409 Let it Bead(BOJ 6567)"
date = 2025-03-04T13:53:45+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2409
https://www.acmicpc.net/problem/6567

https://vjudge.net/problem/POJ-2409
https://vjudge.net/problem/Baekjoon-6567
<!--more-->
## 問題概要
- $C$色のビーズを合計$N$個繋げた円形のネックレスは、回転・反転等を同一視すると何通りあるか？
### 制約
- $CN\leq 32$
## 解法メモ
[[POJ 1286 Necklace of Beads(BOJ 9817, EOlymp 2227)]]では固定だった色の数を変えれば通る。
解法などについてはそちらを参照。

```cpp
#include <cmath>
#include <iostream>
typedef long long ll;
#define rep(i, n) for (ll i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

ll gcd(ll a, ll b) {
    if (a < b) return gcd(b, a);
    if (b == 0) return a;
    return gcd(b, a % b);
}

int main() {
    while (true) {
        int c, s;
        cin >> c >> s;
        if (c == 0 && s == 0) break;
        ll sum = 0, cnt = 0;
        rep(i, s) {
            sum += pow(c, gcd(s, i));
            cnt++;
        }
        if (s % 2 == 0) {
            sum += pow(c, s / 2) * (s / 2);
            cnt += s / 2;
            sum += pow(c, (s - 1) / 2 + 2) * (s / 2);
            cnt += s / 2;
        } else {
            sum += pow(c, (s - 1) / 2 + 1) * s;
            cnt += s;
        }
        cout << sum / cnt << '\n';
    }
    return 0;
}```