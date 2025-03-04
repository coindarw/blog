+++
title = "POJ 3537 Crosses and Crosses(Gym 100078C)"
date = 2025-03-04T13:54:12+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3537
https://codeforces.com/gym/100078/attachments

https://vjudge.net/problem/POJ-3537
https://vjudge.net/problem/Gym-100078C
<!--more-->
## 問題概要
- AliceとBobがゲームを行う。Aliceが先手で交互に手番が回る
- $1\times N$のマス目に、交互にxを書き込んでいき、自分のターンで３つ並ぶと勝ちになる
### 制約
- $1\leq N\leq 2000$

## 解法メモ
- 不偏ゲームなのでGrundy数を考えることができる

- マス目にxを書き込むということは、その場所でマス目を分割することだと考えられる
	- 両者がミスをしない場合，ゲームの最後を除けば既にあるxに隣接させることはない

-  たまに見る「盤面に分裂があるゲーム」
- このような場合は分裂後の各盤面のGrundy数のXORを取ればよい

- `dp[i][j] = 空きマスがi個連続している盤面で、(両端どちらもxがない(j=0)/片端にxがある(j=1)/両端どちらもxがある(j=2))場合のGrundy数` などとしてGrundy数を計算すればよい

## 実装例
コメントアウト部分はメモ化再帰で実装したもの。多分これで正しい答えは出るのだが、POJだとTLが厳しくメモ化再帰だと通らなかったのでDPをしている。i昇順に求めていけばよい
```cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;
#define all(v) v.begin(), v.end()

/*
int memo[2001][3];
int rec(int i, int j) {
    if (i == 0) return 0;
    if (memo[i][j] != -1) return memo[i][j];

    vector<int> nextGrundy;
    if (j == 0) {  // 両端どちらもxが書かれていない(最初のターン)
        for (int k = 0; k <= i - 1; ++k) {
            nextGrundy.push_back(rec(k, 1) ^ rec(i - k - 1, 1));
        }
    } else if (j == 1) {  // 片端にxがある
        for (int k = 0; k <= i - 3; ++k) {
            nextGrundy.push_back(rec(k, 1) ^ rec(i - k - 1, 2));
        }
    } else {  // 両端どちらもxがある
        for (int k = 2; k <= i - 3; ++k) {
            nextGrundy.push_back(rec(k, 2) ^ rec(i - k - 1, 2));
        }
    }
    sort(all(nextGrundy));
    nextGrundy.erase(unique(all(nextGrundy)), nextGrundy.end());

    rep(k, nextGrundy.size()) {
        if (nextGrundy[k] != k) {
            return memo[i][j] = k;
        }
    }
    return memo[i][j] = nextGrundy.size();
}
*/

int dp[2001][3];
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    memset(dp, -1, sizeof(dp));  // DP用配列を-1で初期化
    // memset(memo, -1, sizeof(memo));

    int n;
    cin >> n;

    rep(i, 3) dp[i][0] = dp[i][1] = dp[i][2] = 0;

    vector<int> nextGrundy;
    for (int i = 3; i <= n; ++i) rep(j, 3) {
            int l, r;
            if (j == 0) l = 0, r = i - 1;
            else if (j == 1) l = 0, r = i - 3;
            else l = 2, r = i - 3;
            nextGrundy.clear();

            for (int k = l; k <= r; ++k) {
                if (j == 0) nextGrundy.push_back(dp[k][1] ^ dp[i - k - 1][1]);
                else if (j == 1) nextGrundy.push_back(dp[k][1] ^ dp[i - k - 1][2]);
                else nextGrundy.push_back(dp[k][2] ^ dp[i - k - 1][2]);
            }

            // mexを求める
            sort(all(nextGrundy));
            nextGrundy.erase(unique(all(nextGrundy)), nextGrundy.end());
            rep(k, nextGrundy.size()) {
                if (nextGrundy[k] != k) {
                    dp[i][j] = k;
                    break;
                }
            }
            if (dp[i][j] == -1) dp[i][j] = nextGrundy.size();
        }

    cout << (dp[n][0] != 0 ? 1 : 2) << '\n';
}
```