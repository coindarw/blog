+++
title = "Codeforces 25E Test"
date = 2025-03-04T13:53:05+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://codeforces.com/problemset/problem/25/E

https://vjudge.net/problem/CodeForces-25E
<!--more-->
## 問題概要
- 3つの英小文字からなる空でない文字列が与えられる。それらすべてを部分文字列として含む最短の文字列の長さを求めよ
### 制約
- 3つの文字列の長さの合計は$10^5$を超えない
## 解法メモ
- 3つの文字列のうち、片方がもう一方を部分文字列として含むもの（$s_i=s_j$の場合含む）があった場合、含まれる方について考える必要はないので削除する

- 出てくる文字列の登場する順番を固定してしまえば、最後に登場した文字列に対してできる限り重なる文字数が大きくなるように、貪欲に文字列を並べていけば最短になる。
- あるか重ね方が可能かはZ-AlgorithmやRolling Hashを使えばよい

- 文字列の登場する順番は最大でも6通りなので、全通りに対して上の貪欲法を行った時の最小値を答えればよい

[ABC343G Compress Strings](https://atcoder.jp/contests/abc343/tasks/abc343_g)より少し簡単
## 実装例
Z-Algorithmを使った
```cpp
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
constexpr int inf = 2000'000'000;
#define all(v) begin(v), end(v)
using namespace std;

vector<int> z_algo(string s) {
    int n = s.size();
    vector<int> v(n);
    v[0] = n;
    int l = -1, r = -1;
    for (int i = 1; i < n; ++i) {
        if (l != -1) v[i] = max(min(v[i - l], int(r - i)), 0);
        while (i + v[i] < n && s[v[i]] == s[i + v[i]]) v[i]++;
        if (i + v[i] > r) {
            l = i;
            r = i + v[i];
        }
    }
    return v;
}
bool s_contains_t(const string &s, const string &t) {
    if (s.size() < t.size()) return false;
    vector<int> v = z_algo(t + s);
    return any_of(begin(v) + t.size(), end(v), [&](int x) { return x >= t.size(); });
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    vector<string> s(3);
    rep(i, 3) cin >> s[i];

    // 重複削除
    sort(all(s));
    s.erase(unique(all(s)), end(s));
    int n = s.size();

    // どれかが他の文字列を部分文字列として含む場合、短いほうは削除
    set<string> st(all(s));
    rep(i, n) rep(j, n) {
        if (i == j) continue;
        if (s_contains_t(s[i], s[j])) st.erase(s[j]);
    }
    s = vector<string>(all(st));
    n = s.size();

    vector<int> idx(n);
    iota(all(idx), 0);
    int ans = inf;
    do {
        int res = s[idx[0]].size();
        rep(i, n - 1) {
            int l1 = s[idx[i]].size(), l2 = s[idx[i + 1]].size();
            vector<int> z = z_algo(s[idx[i + 1]] + s[idx[i]]);
            bool f = false;  // 1文字以上重ねて連結できるか
            rep(j, l1) {
                if (z[l2 + j] >= l1 - j) {
                    f = true;
                    res += l2 - (l1 - j);
                    break;
                }
            }
            if (!f) {
                res += l2;
            }
        }
        ans = min(ans, res);
    } while (next_permutation(all(idx)));
    cout << ans << "\n";
}
```