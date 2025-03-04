+++
title = "AOJ 2164 Revenge of the Round Table(BOJ 22623)"
date = 2025-03-04T13:52:57+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://onlinejudge.u-aizu.ac.jp/challenges/search/titles/2164
https://www.acmicpc.net/problem/22623

https://vjudge.net/problem/Aizu-2164
https://vjudge.net/problem/Baekjoon-22623
<!--more-->
## 問題概要
- 2つの国A,Bから合計$N$人が会合のために円卓に座る
- 同じ国の$K$人以上が並んで座らないような配置の数は何通りあるか、回転して一致するものは同一視して$\mod 10^6+3$で数え上げよ
### 制約
- $1\leq N\leq1000$
- $1\leq K\leq1000$

- $K<N$とは限らないことに注意
## 解法メモ

- ポリアの数え上げ定理を使うが、それなりに面倒


- $N\leq K$の時は特に制限がないので、[[POJ 2409 Let it Bead(BOJ 6567)]]と同じになる。以下、$N>K$として考える。


- 「$i$人分ずらしたときに一致する、条件を満たす座り方の数」は、「長さ$\gcd(i, N)$の01列のうち、先頭と末尾がつながっていると考えた時に同じ数が$K$個より多く並ばないものの数」と言い換えることができる（ここでは回転一致は除かなくていい）


- $\gcd(i, N)$の種類数は$N$の約数なので、非常に小さくなる。$N\leq 1000$なので最大でも約数の個数は32個(840の時)となり、それぞれについて$O(N^2)$位かけても間に合うので、$\gcd(i,N)$の値それぞれについて数え上げることにする。
### 「長さ$d$の01列のうち、先頭と末尾がつながっていると考えた時に同じ数が$K$個より多く並ばないものの数」の数え上げ

- 対称性から後で2倍することにして、先頭は1固定であると考える。


- 先頭の1と繋がっている、連続する1の区間は$O(d^2)$パターン考えられる。


- これを固定し、前から$i$個、後ろから$j$個1が続いているとしたときに残りの部分、つまり「先頭も末尾も0である、長さ$d-i-j$の区間」の個数を足し合わせたものが答えになる


- これは例えば `dp[i][j][k] = i番目まで決めた時点でj個kが連続している数列の数` のようなDPで求めることができる
## 実装例

```cpp
#include <bits/stdc++.h>
using ll = long long;
using namespace std;

vector<ll> divisor(ll n) {
    vector<ll> res;
    for (ll i = 1; i * i <= n; ++i) {
        if (n % i == 0) {
            res.push_back(i);
            if (i * i != n) {
                res.push_back(n / i);
            }
        }
    }
    sort(res.begin(), res.end());
    return res;
}

constexpr ll M = 1'000'000 + 3;

ll mod_power(ll x, ll n) {
    if (n == 0) return 1;
    if (n % 2 == 0) {
        ll r = mod_power(x, n / 2);
        r *= r;
        return r % M;
    }
    ll r = mod_power(x, n - 1);
    r *= x;
    return r % M;
}

ll mod_inv(ll x) { return mod_power(x, M - 2); }

int main() {
    while (true) {
        int n, k;
        cin >> n >> k;
        if (n == 0 && k == 0) break;
        if (n <= k) {
            ll ans = 0;
            for (int i = 0; i < n; ++i) ans += mod_power(2, gcd(i, n)), ans %= M;
            ans *= mod_inv(n);
            ans %= M;
            cout << ans << '\n';
            continue;
        }

        auto div = divisor(n);

        // cnt[i] = 周期がdiv[i]のものの数を数え上げる
        vector<ll> cnt(div.size());
        for (int di = 0; di < (int)div.size(); ++di) {
            ll d = div[di];

            // まずは先頭も末尾も0である、長さxの数列の個数　をDPで求める
            // dp[i][j][k] = i番目まで決めた時点でj個kが連続している数列の個数。最初の要素は0とする
            int m = min<int>(d, k);
            vector dp(d + 1, vector(m + 1, vector(2, 0ll)));
            dp[1][1][0] = 1;
            for (int i = 1; i <= d - 1; ++i)
                for (int j = 0; j < m + 1; ++j)
                    for (int k = 0; k < 2; ++k) {
                        if (j + 1 <= m && (i != d - 1)) dp[i + 1][j + 1][k] += dp[i][j][k], dp[i + 1][j + 1][k] %= M;
                        dp[i + 1][1][1 - k] += dp[i][j][k], dp[i + 1][1][1 - k] %= M;
                    }

            // これがDPで求めたかった「先頭も末尾も0である、長さxの数列の個数」
            vector sum(d + 1, 0ll);
            for (int i = 1; i <= d; ++i)
                for (int j = 1; j <= min<int>(d - 1, k); ++j) sum[i] += dp[i][j][0], sum[i] %= M;

            // 「長さdの01列のうち、先頭と末尾がつながっていると考えた時に同じ数がK個より多く並ばないもの」の数
            ll res = 0;
            for (int i = 1; i <= m; ++i) {  // 先頭からi個、末尾からj個が1であるとする
                for (int j = 0; i + j <= min<int>(d - 1, k); ++j) {
                    // 仮定からすべて1にはならないのでi,jが動くのはこの範囲
                    res += sum[d - i - j];
                    res %= M;
                }
            }
            cnt[di] += res * 2;  // 01反転したものもvalidなので2倍
            cnt[di] %= M;
        }

        ll ans = 0;
        for (int i = 0; i < n; ++i) {  // ポリアの数え上げ定理
            ll g = gcd(i, n);
            ans += cnt[lower_bound(div.begin(), div.end(), g) - div.begin()];
        }
        ans *= mod_inv(n);
        ans %= M;
        cout << ans << '\n';
    }
}
```