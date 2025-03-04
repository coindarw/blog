+++
title = "POJ 2407 Relatives(UVa 10299)"
date = 2025-03-04T13:53:43+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2407
https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=1240

https://vjudge.net/problem/POJ-2407
https://vjudge.net/problem/UVA-10299
<!--more-->
## 問題概要
正整数$N$が与えられる。$N$未満の整数$a$で、$N$と互いに素なものの個数を数え上げよ
### 制約
- マルチテストケース
- $N\leq 10^9$
## 解法メモ
オイラーのφ関数の定義そのまま。
[[POJ 1284 Primitive Roots]]の入力だけ変えて貼ると通る

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
    while (true) {
        scanf("%d", &p);
        if (p == 0) break;
        printf("%d\n", euler_phi(p));
    }
}
```
