+++
title = "POJ 2975 Nim(BOJ 7685)"
date = 2025-03-04T13:53:54+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2975
https://www.acmicpc.net/problem/7685

https://vjudge.net/problem/POJ-2975
https://vjudge.net/problem/Baekjoon-7685
<!--more-->
## 問題概要
- $N\ (\leq 1000)$個の山にある石の数$A_i\leq10^9$が与えられ、その盤面でNimの自分の手番が回ってきたとする。その手番で取れる行動のうち、勝利できる手の個数を数え上げよ
## 解法メモ
- 与えられた状態での総XOR$\bigoplus_{i=1}^N A_i=0$の時はそもそも負けが確定しているので0を出力する。
- そうでない場合、勝利できる手＝操作後の総XORが０になる手　となる。
- 取る山を決めた時、その山の石の個数を$(\bigoplus_{i=1}^N A_i)\oplus A_i$にすれば総XORを0にすることができ、これ以外の値で0にすることはできない。
- 各$i$についてこの値が$A_i$以下であるものを数えればよい
```cpp
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int n;
        cin >> n;
        if (n == 0) break;
        vector<int> v(n);
        rep(i, n) cin >> v[i];

        int all_xor = 0;
        rep(i, n) all_xor ^= v[i];

        if (all_xor == 0) {
            cout << 0 << '\n';
            continue;
        }

        int ans = 0;
        rep(i, n) if ((all_xor ^ v[i]) <= v[i]) ans++;

        cout << ans << '\n';
    }
    return 0;
}
```