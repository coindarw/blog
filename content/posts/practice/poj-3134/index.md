+++
title = "POJ 3134 Power Calculus(AOJ 1271)"
date = 2025-03-04T13:53:56+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++


http://poj.org/problem?id=3134
https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1271
https://www.spoj.com/problems/YOKOF/en/

https://vjudge.net/problem/POJ-3134
https://vjudge.net/problem/Aizu-1271
https://vjudge.net/problem/SPOJ-YOKOF
※POJのジャッジだと通っていない
<!--more-->
## 問題概要
- 正整数の乗算・除算ができるときに、正整数$x$の$n$乗を高速に求めたい
	- （繰り返し二乗法のようにすれば$\log_2$回程度でできるが、これは最小ではない
- $n$が与えられるので、$x^n$を計算する際の最小の乗算・除算の回数を求めよ

### 制約
$1\leq n\leq 1000$
## 解法メモ
Project Eulerに似た問題があった気がする

- 枝刈り全探索を実装するが、小手先の枝刈りをごにょごにょしていたら通ってしまった
- 最適化込みだと1秒くらいで実行できるが、POJは最適化オプションを付けてくれないので通っていない


## 実装例
現在までに見た最大値を使って$N$が作れそうなら作り、無理なら再帰するといった適当な枝刈り。おそらく参考にはならない
```cpp
#include <algorithm>
#include <cstdio>
#include <cstdlib>
#include <ctime>
#include <iostream>
#include <map>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;
#define all(v) v.begin(), v.end()
#define rall(v) v.rbegin(), v.rend()

vector<int> hashes;

vector<int> used;
map<int, int> memo;
vector<int> uppers;
vector<bool> seen;
int n;

int g(int x, int d = 0) {  // 普通の繰り返し二乗法を上界として使う
    if (x <= 1) return d;
    if (x % 2 == 0) return g(x / 2, d + 1);
    return g(x - 1, d + 1);
}

int rec(int upper) {
    if (used.size() >= upper) return upper;
    int maxUsed = *max_element(all(used));

    // 枝刈り。最大値を2倍していってそれだけでnを作れるなら作る
    int x = maxUsed, c = 0;
    int osz = used.size();
    while (x < n) {
        x *= 2;
        used.push_back(x);
        c++;
    }
    int usz = used.size();
    if (x == n) {
        used.resize(osz);
        return usz - 1;
    } else {
        if (c != 0) {
            rep(i, usz - 1) {
                if (x / 2 + used.at(i) == n) {
                    used.resize(osz);
                    return usz - 1;
                }
            }
        }
        rep(i, usz) {
            if (x - used.at(i) == n) {
                used.resize(osz);
                return usz;
            }
        }
        if (usz - 1 >= upper) {
            used.resize(osz);
            return upper;
        }
        used.resize(osz);
    }

    // 再帰先の候補を列挙
    int res = upper;
    vector<int> candidates;

    for (size_t i = 0; i < used.size(); i++) {
        int x = used[i];
        int sum = x + maxUsed;
        int dif = maxUsed - x;
        if (sum == n || dif == n) {
            return used.size();
        }
        if (sum != 0 && sum < 2 * n && !seen[sum]) {
            uppers[sum] = min<int>(uppers[sum], used.size());
            candidates.push_back(sum);
        }
        if (dif != 0 && !seen[dif]) {
            uppers[dif] = min<int>(uppers[dif], used.size());
            candidates.push_back(dif);
        }
    }
    sort(rall(candidates));
    candidates.erase(unique(all(candidates)), candidates.end());

    // バックトラック
    for (size_t i = 0; i < candidates.size(); i++) {
        int next = candidates[i];
        used.push_back(next);
        seen[next] = true;
        int r = rec(res);
        if (res > r) res = r;
        seen[next] = false;
        used.pop_back();
    }
    return res;
}

int main() {
    uppers.resize(2049);
    rep(i, 2049) uppers[i] = g(i);
    while (true) {
        cin >> n;
        if (n == 0) break;
        if (n == 1) {
            cout << 0 << endl;
            continue;
        }
        seen = vector<bool>(n * 2 + 1);
        seen[1] = true;
        seen[2] = true;
        used.clear();
        memo.clear();
        used.push_back(1);
        used.push_back(2);
        cout << rec(uppers[n]) << endl;
    }
}
```