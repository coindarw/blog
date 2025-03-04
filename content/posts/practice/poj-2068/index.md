+++
title = "POJ 2068 Nim (AOJ 1230, BOJ 9157)"
date = 2025-03-04T13:53:33+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2068
https://onlinejudge.u-aizu.ac.jp/challenges/search/titles/1230
https://www.acmicpc.net/problem/9157

https://vjudge.net/problem/POJ-2068
https://vjudge.net/problem/Aizu-1230
https://vjudge.net/problem/Baekjoon-9157
<!--more-->
## 問題概要
(面倒なので0-indexedにしている)

- 以下の形式で入力が与えられる
	- $N\ S\ M_0\ M_1\ \cdots\ M_{2N-1}$
- チームA,BがNimを行う。最初、山には石が$S$個ある。
- $i\ (\geq 0)$手目では、
	- $i$が奇数の場合チームAが、1個以上$M_{i\bmod{2N}}$個以下の石を取り除く
	- $i$が偶数の場合チームBが、1個以上$M_{i\bmod{2N}}$個以下の石を取り除く
- 最後の1個の石を取り除いたチームの負けである（普通と逆なのに注意）
- チームAが勝利戦略を持つか判定せよ
### 制約
- $1\leq n\leq 10$
- $1\leq M_i \leq 16$
- $1\leq S\leq 2^{13}$
## 解法メモ
(現在誰の手番か, 現在の石の数)を持ってメモ化再帰すればよい
## 実装例
```cpp
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int n, s;
vector<int> m;
vector<vector<int> > memo;
bool rec(int turn, int num) {
    if (num == 1) return false;
    if (memo[turn][num] != -1) return memo[turn][num];

    for (int i = 1; i <= m[turn]; ++i) {
        if (num - i <= 0) break;
        if (!rec((turn + 1) % (2 * n), num - i)) return memo[turn][num] = true;
    }
    return memo[turn][num] = false;
}

int main() {
    while (true) {
        cin >> n;
        if (n == 0) break;
        cin >> s;
        m.resize(2 * n);
        rep(i, 2 * n) cin >> m[i];

        memo.assign(2 * n, vector<int>(s + 1, -1));
        cout << rec(0, s) << "\n";
    }
}
```