+++
title = "POJ 3250 Bad Hair Day(luogu P2866)"
date = 2025-03-04T13:53:59+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3250
https://www.luogu.com.cn/problem/P2866

https://vjudge.net/problem/POJ-3250
https://vjudge.net/problem/%E6%B4%9B%E8%B0%B7-P2866
<!--more-->
## 問題概要
長さ$N$の数列$h_i$が与えられる。以下の値を求めよ。
$$
\sum_{1\leq i<j\leq N}f(i,j)
$$
ただし、
$$
f(i,j)=\left\{
\begin{array}{ll}
1 & \mathrm{if}\quad h_i>\max(h_{i+1},h_{i+2},\cdots, h_{j}) \\
0 & \mathrm{otherwise}
\end{array}
\right.
$$
とする。
### 制約
$1\leq h_i\leq 10^9$

## 解法メモ
- 幾つか解法は考えられ、例えばセグ木＋二分探索などでも出来るが、stackを使えば線形時間で解くことができる。

- $f(i,j)$の$j$を固定するような感じで見ていくとよい。

- $h_j$を左から順にみていき、$j$番目まで見た時に、$i$の候補としてあり得るものだけをスタックに格納していく。
- これは狭義単調減少になり、$h_j$を格納する時に末尾からそれ以上のものを削除していけばよい。

## 実装例
```cpp
#include <iostream>
#include <stack>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n;
    cin >> n;
    vector<int> h(n);
    rep(i, n) cin >> h[i];

    stack<int> st;
    ll ans = 0;
    rep(j, n) {
        while (!st.empty() && st.top() <= h[j]) st.pop();
        ans += st.size();
        st.push(h[j]);
    }

    cout << ans << "\n";
}
```