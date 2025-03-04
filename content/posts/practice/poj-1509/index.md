+++
title = "POJ 1509 Glass Beads(BOJ 3492,EOlymp 2912,SPOJ BEADS)"
date = 2025-03-04T13:53:23+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1509
https://www.acmicpc.net/problem/3492
https://basecamp.eolymp.com/en/problems/2912
https://www.spoj.com/problems/BEADS/en/

https://vjudge.net/problem/POJ-1509
https://vjudge.net/problem/Baekjoon-3492
https://vjudge.net/problem/EOlymp-2912
https://vjudge.net/problem/SPOJ-BEADS
<!--more-->
## 問題概要
- 文字列$S$が与えられる
- その巡回シフトの中で辞書順最小なのはどこで切ったときか答えよ
- 複数あるなら切る場所の番号が最小のところ
### 制約
- $|S|\leq 10^4$
## 解法メモ
- ※ 文字列$S$のSAを求めると、空文字列$\varepsilon$はカウントしないで$0,\cdots,|S|-1$の順列が返ってくるとする

- 辞書順最小のものを見つけるだけなら$S$を2つ繋げてSAを求めて最初に出てくる$N$未満の要素を求めればよい

- 今回はタイブレークまで求められているため、$S+S$ に適当な番兵を入れた文字列のSAを求め、最初に出てくる$N$未満の要素を求めればよい

## 解法メモ
asciiコードで`z`の次は`{`なので$S+S$にこれを連結した文字列のSAを求めている
```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (ll i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

vector<int> sa_is(vector<int> v, int upper) {
    // 1 <= v[i] <= upper
    if (v.size() == 0) return vector<int>();
    else if (v.size() == 1) return vector<int>(1, 0);
    else if (v.size() == 2) {
        vector<int> res(2, 0);
        if (v[0] < v[1]) res[1] = 1;
        else res[0] = 1;
        return res;
    }
    v.push_back(0);  // sentinel
    const int n = v.size();

    vector<int> bl(upper + 1), br(upper + 1);  // bucket range
    for (int i = 0; i < n; ++i) br[v[i]]++;
    for (int i = 1; i <= upper; ++i) br[i] += br[i - 1];
    for (int i = 1; i <= upper; ++i) bl[i] = br[i - 1];

    vector<int> is_l(n);
    vector<int> lms, sa(n, -1), lms_ord(n);  // lms_ord[i] := 0 -> not lms, 1~ -> 1-indexed
    for (int i = n - 2; i >= 0; --i) is_l[i] = (v[i] == v[i + 1]) ? is_l[i + 1] : (v[i] > v[i + 1]);
    for (int i = 1; i < n; ++i) {
        if (!is_l[i] && is_l[i - 1]) {
            sa[--br[v[i]]] = i;
            lms_ord[i] = ~int(lms.size());
            lms.push_back(i);
        }
    }

    for (int i = 0; i < upper; ++i) br[i] = bl[i + 1];
    br[upper] = n;

    for (int i = 0; i < n; ++i)
        if (sa[i] > 0 && is_l[sa[i] - 1]) sa[bl[v[sa[i] - 1]]++] = sa[i] - 1;
    for (int i = 1; i <= upper; ++i) bl[i] = br[i - 1];
    for (int i = 1; i < n; i++)
        if (sa[i] > -1 && !is_l[sa[i]]) sa[i] = -1;
    for (int i = n - 1; i >= 1; i--)
        if (sa[i] > 0 && !is_l[sa[i] - 1]) sa[--br[v[sa[i] - 1]]] = sa[i] - 1;
    for (int i = 0; i < upper; ++i) br[i] = bl[i + 1];
    br[upper] = n;

    vector<int> lms_substr_sorted(lms.size());
    int cnt = 0;
    for (int i = 0; i < n; ++i)
        if (sa[i] > -1 && lms_ord[sa[i]]) lms_substr_sorted[cnt++] = sa[i];

    // same lms_substr -> same rank
    vector<int> ord(lms.size());
    ord[0] = 1;
    for (int i = 0; i < int(lms.size()) - 1; ++i) {
        int l1 = lms_substr_sorted[i], l2 = lms_substr_sorted[i + 1];
        if (l1 > l2) swap(l1, l2);
        if (l2 == n - 1) ord[i + 1] = ord[i] + 1;
        else {
            int p1 = l1, p2 = l2;
            bool f = true;
            while (p1 <= lms[~lms_ord[l1] + 1] && p2 < n)
                if (v[p1] == v[p2]) ++p1, ++p2;
                else {
                    f = false;
                    break;
                }
            ord[i + 1] = f ? ord[i] : ord[i] + 1;
        }
    }
    vector<int> va(lms.size());  // make array of appearance order

    for (int i = 0; i < int(lms.size()); ++i) va[~lms_ord[lms_substr_sorted[i]]] = ord[i];

    vector<int> lms_sorted = sa_is(va, ord.back());

    // place lms at correct position
    fill(sa.begin(), sa.end(), -1);
    for (int i = lms.size() - 1; i >= 0; i--) sa[--br[v[lms[lms_sorted[i]]]]] = lms[lms_sorted[i]];
    for (int i = 0; i < upper; ++i) br[i] = bl[i + 1];
    br[upper] = n;
    for (int i = 0; i < n; ++i)
        if (sa[i] > 0 && is_l[sa[i] - 1]) sa[bl[v[sa[i] - 1]]++] = sa[i] - 1;
    for (int i = 1; i < n; i++)
        if (sa[i] > -1 && !is_l[sa[i]]) sa[i] = -1;
    for (int i = n - 1; i >= 1; i--)
        if (sa[i] > 0 && !is_l[sa[i] - 1]) sa[--br[v[sa[i] - 1]]] = sa[i] - 1;
    sa.erase(sa.begin());
    return sa;
}
vector<int> sa_is(string s) {
    vector<int> v(s.size());
    for (int i = 0; i < int(s.size()); i++) v[i] = s[i];
    return sa_is(v, 255);
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int q;
    cin >> q;
    while (q--) {
        string s;
        cin >> s;
        int n = s.size();

        s += s;
        s += char('z' + 1);
        vector<int> sa = sa_is(s);

        ll ans = 0;
        rep(i, 2 * n) {
            if (sa[i] < n) {
                cout << sa[i] + 1 << "\n";
                break;
            }
        }
    }
}
```