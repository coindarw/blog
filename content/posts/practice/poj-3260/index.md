+++
title = "POJ 3260 The Fewest Coins(BOJ 6205,Luogu P2851)"
date = 2025-03-04T13:54:01+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3260
https://www.acmicpc.net/problem/6205
https://www.luogu.com.cn/problem/P2851

https://vjudge.net/problem/POJ-3260
https://vjudge.net/problem/Baekjoon-6205
https://vjudge.net/problem/%E6%B4%9B%E8%B0%B7-P2851
<!--more-->
## 問題概要
- 通貨システムには$V_1, \cdots, V_N$セントの$N$種類の異なるコインがある。
- 店で$T$セントのものを購入したい。
- それぞれのコインを$C_1, \cdots, C_N$枚持っている。

- 支払いに使用するコインの数＋おつりで受け取るコインの数の最小値を答えよ。
	- ちょうどのお釣りを受け取れない場合は報告。
- ただし、店はすべてのコインを無限枚持ち、最小枚数でお釣りを払う。

### 制約
- $1\leq N\leq 100$
- $1\leq T\leq 10^4$
- $0\leq C_i\leq 10^4$
- $1\leq V_i\leq 120$
## 解法メモ
- 支払いに使用するコインの数
	- 持っているコインで$i$円払う時の使うコインの数の最小値を個数制限ありナップサックで求める
- お釣りで受け取るコインの数
	- 枚数無制限で$i$円払う時の受け取るコインの数の最小値を個数制限なしナップサックで求める
- を求めて、支払う金額を30000まで全探索したら通る

- 例えば$2T$まで全探索だと100円玉と90円玉しかない時に10円のものを買おうとするときなどに壊れる。良い上界が存在しそうだがわからず
	- [DEC06 CONTEST Analysis and Data (archive.org)](https://web.archive.org/web/20181231044524/http://contest.usaco.org:80/DEC06.htm)解説のアーカイブなども残っていないので残念

- 個数制限ありナップサックについては[前に書いた記事](https://coindarw.github.io/blog/posts/knapsack-with-limitations/)を参照
## 実装例
```cpp
#include <algorithm>
#include <iostream>
#include <stack>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(1e9)
using namespace std;

template <typename T, T (*op)(T, T)>
struct SWAG {
    stack<T, vector<int> > st1, st2, raw;

    void push(T x) {
        if (st1.empty()) st1.push(x);
        else st1.push(op(st1.top(), x));
        raw.push(x);
    }

    void move() {
        if (st2.empty()) {
            while (!st1.empty()) {
                st1.pop();
                if (st2.empty()) st2.push(raw.top());
                else st2.push(op(raw.top(), st2.top()));
                raw.pop();
            }
        }
    }

    void pop() {
        move();
        st2.pop();
    }

    T prod() {
        if (st2.empty()) return st1.top();
        if (st1.empty()) return st2.top();
        return op(st2.top(), st1.top());
    }

    int size() { return st1.size() + st2.size(); }
};

int op(int a, int b) { return min(a, b); }

vector<int> knapsack_with_limitation(const vector<pair<int, int> > &items, int W) {
    // minimize
    const int N = items.size();
    vector<vector<int> > dp = vector<vector<int> >(N + 1, vector<int>(W + 1, inf));
    dp[0][0] = 0;
    for (int i = 1; i <= N; ++i) {
        int wi = items[i - 1].first, mi = items[i - 1].second;
        int vi = 1;
        for (int a = 0; a < wi; ++a) {
            SWAG<int, op> swag;
            for (int j = a; j <= W; j += wi) {
                swag.push(dp[i - 1][j] - j / wi * vi);
                if (swag.size() > mi + 1) swag.pop();
                dp[i][j] = swag.prod() + vi * (j / wi);
            }
        }
    }
    return dp.back();
}
vector<int> knapsack_without_limitation(const vector<int> &items, int W) {
    // minimize
    const int N = items.size();
    vector<vector<int> > dp = vector<vector<int> >(N + 1, vector<int>(W + 1, inf));
    dp[0][0] = 0;
    for (int i = 1; i <= N; ++i) {
        int vi = 1, wi = items[i - 1];
        for (int j = 0; j <= W; ++j) {
            dp[i][j] = dp[i - 1][j];
            if (j >= wi) dp[i][j] = min(dp[i][j], dp[i][j - wi] + vi);
        }
    }
    return dp.back();
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, t;
    cin >> n >> t;
    vector<int> v(n), c(n);
    rep(i, n) cin >> v[i];
    rep(i, n) cin >> c[i];

    vector<pair<int, int> > items;
    rep(i, n) items.push_back(make_pair(v[i], c[i]));

    const int T = 30000;
    vector<int> dp1 = knapsack_with_limitation(items, T + 1);
    vector<int> dp2 = knapsack_without_limitation(v, T + 1);

    int ans = inf;
    rep(i, T + 1 - t) ans = min(ans, dp1[t + i] + dp2[i]);

    if (ans == inf) ans = -1;
    cout << ans << "\n";
}
```