+++
title = "POJ 1854 Evil Straw Warts Live(BOJ 4309,UVA 10716)"
date = 2025-03-04T13:53:26+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1854
https://www.acmicpc.net/problem/4309
https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=1657

https://vjudge.net/problem/POJ-1854
https://vjudge.net/problem/Baekjoon-4309
https://vjudge.net/problem/UVA-10716
<!--more-->
## 問題概要
- 文字列$s$が与えられる。隣接swapによって回文にできるか判定、できる場合は最小回数を出力
### 制約
- $|s|\leq 8000$
## 解法メモ
- 回文にできる条件は「奇数個存在する値が1種類以下であること」。これはよく見る気がする

- 最終的にどのような形になるかだが、これは左半分を貪欲に決めてよい
- $s$の各文字$s_i$を順番に見て、登場回数が全体の半分を超えるまで取っていく（実装を参照）

- あとは$p_i =$ 「文字$s_i$が最終的につくる回文で何番目に出てくるか」の反転数が答えになる

## 実装例
BITで実装したもの。最初に書いた下の雑実装マージソートより数倍速い
```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) v.begin(), v.end()
using namespace std;

template <typename T>
struct BIT {
    int n;
    vector<T> data;
    BIT(int n) : n(n), data(n + 1, 0) {}
    BIT() {}
    void add(int i, T x = 1) {
        for (i++; i <= n; i += (i & -i)) data[i] += x;
    }
    T sum(int i) {
        T s(0);
        for (i++; i; i -= (i & -i)) s += data[i];
        return s;
    }
    T sum(int l, int r) { return sum(r - 1) - sum(l - 1); }
    T get(int i) { return sum(i) - sum(i - 1); }

    int lower_bound(T w) {
        if (w <= 0) {
            return 0;
        }
        int x = 0, r = 1;
        while (r < n) r <<= 1;
        for (int len = r; len > 0; len >>= 1) {
            if (x + len <= n && data[x + len] < w) {
                w -= data[x + len];
                x += len;
            }
        }
        return x;
    }
};

const vector<int> *ptr;
bool compare(int a, int b) { return (ptr->at(a) == ptr->at(b)) ? (a > b) : (ptr->at(a) > ptr->at(b)); }
template <typename T>
long long inversion(const vector<T> &v) {
    const int n = v.size();
    BIT<int> bit(n);
    vector<int> idx(n);
    for (int i = 0; i < n; ++i) idx[i] = i;
    ptr = &v;
    sort(idx.begin(), idx.end(), compare);
    long long ans = 0;
    for (int i = 0; i < n; ++i) {
        int id = idx[i];
        ans += bit.sum(0, id);
        bit.add(id, 1);
    }
    return ans;
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int q;
    cin >> q;
    while (q--) {
        string s;
        cin >> s;

        vector<int> cnt(26, 0);
        rep(i, s.size()) cnt[s[i] - 'a']++;

        int oddCnt = 0;
        rep(i, 26) if (cnt[i] % 2 == 1) oddCnt++;
        if (oddCnt > 1) {
            cout << "Impossible\n";
            continue;
        }

        string t;  // 最終的に作る文字列
        {
            vector<int> tmp(26);
            rep(i, s.size()) {
                // s[i]の現時点での登場回数が半分以下ならtに追加
                tmp[s[i] - 'a']++;
                if (tmp[s[i] - 'a'] <= cnt[s[i] - 'a'] / 2) t.push_back(s[i]);
            }
        }
        // 後ろ半分を繋げて回文にする
        string u = t;
        reverse(all(u));
        if (s.size() % 2 == 1) {  // 奇数の場合は中央の文字を追加
            rep(i, 26) {
                if (cnt[i] % 2 == 1) {
                    t.push_back(i + 'a');
                    break;
                }
            }
        }
        t += u;

        // sの各文字が最終的には何番目に登場するかの順列を求める
        vector<vector<int> > idx(26);
        {
            vector<int> tmp(26);
            rep(i, t.size()) {
                idx[t[i] - 'a'].push_back(i);
                tmp[t[i] - 'a']++;
            }
        }
        vector<int> p(s.size());
        {
            vector<int> tmp(26);
            rep(i, s.size()) {
                p[i] = idx[s[i] - 'a'][tmp[s[i] - 'a']];
                tmp[s[i] - 'a']++;
            }
        }

        // pの反転数が答え
        cout << inversion(p) << "\n";
    }
}
```

マージソートで反転数を求めている。実装が雑なのでかなり低速
```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) v.begin(), v.end()
using namespace std;

ll ans = 0;

template <typename T>
void merge(vector<T> &v1, vector<T> &v2, vector<T> &v) {
    size_t i1 = 0, i2 = 0;
    while (i1 < v1.size() || i2 < v2.size()) {
        if (i2 == v2.size() || (i1 < v1.size() && (v1[i1] < v2[i2]))) {
            v.push_back(v1[i1]);
            i1++;
        } else {
            v.push_back(v2[i2]);
            i2++;
            ans += v1.size() - i1;
        }
    }
}
template <typename T>
void merge_sort(vector<T> &v) {
    if (v.size() <= 1) return;
    vector<T> u(v.begin(), v.begin() + v.size() / 2);
    vector<T> w(v.begin() + v.size() / 2, v.end());
    merge_sort(u);
    merge_sort(w);
    v.clear();
    merge(u, w, v);
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int q;
    cin >> q;
    while (q--) {
        string s;
        cin >> s;

        vector<int> cnt(26, 0);
        rep(i, s.size()) cnt[s[i] - 'a']++;

        int oddCnt = 0;
        rep(i, 26) if (cnt[i] % 2 == 1) oddCnt++;
        if (oddCnt > 1) {
            cout << "Impossible\n";
            continue;
        }

        string t;  // 最終的に作る文字列
        {
            vector<int> tmp(26);
            rep(i, s.size()) {
                // s[i]の現時点での登場回数が半分以下ならtに追加
                tmp[s[i] - 'a']++;
                if (tmp[s[i] - 'a'] <= cnt[s[i] - 'a'] / 2) t.push_back(s[i]);
            }
        }
        // 後ろ半分を繋げて回文にする
        string u = t;
        reverse(all(u));
        if (s.size() % 2 == 1) {  // 奇数の場合は中央の文字を追加
            rep(i, 26) {
                if (cnt[i] % 2 == 1) {
                    t.push_back(i + 'a');
                    break;
                }
            }
        }
        t += u;

        // sの各文字が最終的には何番目に登場するかの順列を求める
        vector<vector<int> > idx(26);
        {
            vector<int> tmp(26);
            rep(i, t.size()) {
                idx[t[i] - 'a'].push_back(i);
                tmp[t[i] - 'a']++;
            }
        }
        vector<int> p(s.size());
        {
            vector<int> tmp(26);
            rep(i, s.size()) {
                p[i] = idx[s[i] - 'a'][tmp[s[i] - 'a']];
                tmp[s[i] - 'a']++;
            }
        }

        // pの反転数が答え
        ans = 0;
        merge_sort(p);
        cout << ans << "\n";
    }
}
```
