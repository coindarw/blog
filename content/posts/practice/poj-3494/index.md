+++
title = "POJ 3494 Largest Submatrix of All 1’s"
date = 2025-03-04T13:54:05+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3494

https://vjudge.net/problem/POJ-3494
<!--more-->
## 問題概要
- $M$行$N$列の01行列が与えられる。全ての要素が1の部分行列で、要素数最大のものの要素数はいくつか
	- この問題での「部分行列（Submatrix）」は行列のある長方形領域を指しそう
### 制約
- $1\leq N,M\leq 2000$
## 解法メモ
- [AOJ Course DPL_3_B Largest Rectangle](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DPL_3_B)とほぼ同じ問題
- 各行ごとに、「そこからいくつ1が連続しているか」をDPで求めてからその中の最大長方形をヒストグラム内最大長方形問題として解く

## 実装例
POJだとcin,coutだとTLがきついのでscanf, printfを使うのがよい
```cpp
#include <cstdio>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

// ヒストグラムを与えると、h[i]を左右にどこまで伸ばせるか（閉区間）を返す
template <typename T>
vector<pair<int, int> > largest_rectangle_in_histogram(vector<T> &h) {
    const int n = h.size();
    vector<pair<int, int> > res(n);
    vector<int> st;
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && h[st.back()] >= h[i]) st.pop_back();
        res[i].first = st.empty() ? 0 : st.back() + 1;
        st.push_back(i);
    }
    st.clear();
    for (int i = n - 1; i >= 0; --i) {
        while (!st.empty() && h[st.back()] >= h[i]) st.pop_back();
        res[i].second = st.empty() ? n - 1 : st.back() - 1;
        st.push_back(i);
    }
    return res;
}

int main() {
    int n, m;
    while (scanf("%d %d", &n, &m) != EOF) {
        vector<vector<int> > a(n, vector<int>(m));
        rep(i, n) rep(j, m) scanf("%d", &a[i][j]);

        rep(i, n) {
            if (i == 0) continue;
            rep(j, m) {
                if (a[i][j] == 1) a[i][j] += a[i - 1][j];
            }
        }

        int ans = 0;
        rep(i, n) {
            vector<pair<int, int> > res = largest_rectangle_in_histogram(a[i]);
            rep(j, m) ans = max(ans, a[i][j] * (res[j].second - res[j].first + 1));
        }
        printf("%d\n", ans);
    }
}
```