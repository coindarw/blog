+++
title = "Codeforces 123D String"
date = 2025-03-04T13:53:02+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://codeforces.com/problemset/problem/123/D

https://vjudge.net/problem/CodeForces-123D
<!--more-->
## 問題概要
- 文字列$x,y$に対して$F(x, y)$を次のように定義する
	- $x$の中で$y$が登場する位置のリスト$L$を作った時、その空でない区間の個数
	- 別の言い方をすると、文字列$y$が文字列$x$の（連続）部分文字列として登場する回数を$G(x,y)$としたとき、$F(x, y)=\dfrac{G(x,y)(G(x,y)+1)}2$である

- 文字列$S$が与えられるので、以下の値を求めよ
$$
\sum_{xはSの部分文字列}F(S,x)
$$

### 制約
$1\leq |S| \leq 10^5$
## 解法メモ
- $S$の中で部分文字列$x$が登場する位置のリスト$L$の、長さ1の区間の個数の合計は$|S|(|S|-1)/1$個
- 以下、リストの長さ2以上の区間の個数を数えることにする。

- LCP配列を使って数え上げをする
- 左端が$SA[i]$である長さ$k$の部分列が何回足されるかを考える
	- これはLCP配列$LA$の中の$LA[i]$を左端とする区間で、区間minが$k$以上となる最長区間の長さである
- 計算量を気にしなければ、同じ$k$の区間をまとめて足していくことにすると、以下のように、LCP配列内の長方形領域を足していって答えを求めることができる
```python
for i in range(len(la)):
    rangeMin = la[i]
    for j in range(i, len(la)):
        rangeMin = min(rangeMin, la[j])
        ans += rangeMin
```

- これを高速に求めることができればよい
	- BITやstd::set等を使い、ある要素が最小値になる区間がどこまでかを二分探索することで求めることができる。
	- [AGC005B Minimum Sum](https://atcoder.jp/contests/agc005/tasks/agc005_b)がこの部分だけやる問題
	- スタックを使ってヒストグラム内最大長方形問題のような感じにすると線形時間でも解ける
## 実装例
setを使って$O(|S|\log |S|)$で解いたもの。
下に線形時間で解いたものも載せている
```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <numeric>
#include <set>
#include <vector>

typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
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
vector<int> lcp(const string& s, const vector<int>& sa) {
    assert(sa.size() == s.size());
    vector<int> rank(s.size()), _lcp(s.size() - 1);
    const int n = s.size();
    for (int i = 0; i < n; i++) rank[sa[i]] = i;
    int h = 0;
    for (int i = 0; i < n; i++) {
        if (rank[i] == 0) continue;
        int j = sa[rank[i] - 1];
        if (h > 0) h--;
        for (; j + h < n && i + h < n; h++)
            if (s[j + h] != s[i + h]) break;
        _lcp[rank[i] - 1] = h;
    }
    return _lcp;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    string s;
    cin >> s;
    int n = s.size();
    vector<int> sa = sa_is(s);
    vector<int> la = lcp(s, sa);

    ll ans = s.size() * ll(s.size() + 1) / 2;

	// Σ_l Σ_r min(la[l..r]) を求める
    set<int> st = {-1, n - 1};
    vector<int> idx(n - 1);
    iota(idx.begin(), idx.end(), 0);
    stable_sort(idx.begin(), idx.end(), [&](int a, int b) { return la[a] < la[b]; });
    for (int i = 0; i < n - 1; ++i) {
        int id = idx[i];
        auto ri = st.lower_bound(id), li = prev(ri);
        ll ln = id - *li, rn = *ri - id;
        ans += ln * rn * la[id];
        st.insert(id);
    }

    cout << ans << endl;
}
```


stackを使って線形時間で解いたもの
```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <numeric>
#include <set>
#include <vector>

typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
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
vector<int> lcp(const string& s, const vector<int>& sa) {
    assert(sa.size() == s.size());
    vector<int> rank(s.size()), _lcp(s.size() - 1);
    const int n = s.size();
    for (int i = 0; i < n; i++) rank[sa[i]] = i;
    int h = 0;
    for (int i = 0; i < n; i++) {
        if (rank[i] == 0) continue;
        int j = sa[rank[i] - 1];
        if (h > 0) h--;
        for (; j + h < n && i + h < n; h++)
            if (s[j + h] != s[i + h]) break;
        _lcp[rank[i] - 1] = h;
    }
    return _lcp;
}

template <typename T>
vector<pair<int, int> > largest_rectangle_in_histogram(const vector<T>& h) {
    const int n = h.size();
    vector<pair<int, int> > res(n);
    vector<int> st;
    for (int i = 0; i < n; ++i) {
        while (!st.empty() && h[st.back()] > h[i]) st.pop_back();
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
ll range_min_sum(const vector<int>& v) {
    const int n = v.size();
    vector<pair<int, int> > lr = largest_rectangle_in_histogram<int>(v);
    ll res = 0;
    rep(i, n) {
        ll ln = i - lr[i].first + 1, rn = lr[i].second - i + 1;
        res += ln * rn * v[i];
    }
    return res;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    string s;
    cin >> s;
    int n = s.size();
    vector<int> sa = sa_is(s);
    vector<int> la = lcp(s, sa);

    ll ans = s.size() * ll(s.size() + 1) / 2;
    ans += range_min_sum(la);
    cout << ans << endl;
}
```