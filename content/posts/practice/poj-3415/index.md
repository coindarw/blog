+++
title = "POJ 3415 Common Substrings"
date = 2025-03-04T13:54:03+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3415

https://vjudge.net/problem/POJ-3415
<!--more-->
## 問題概要
- 文字列$A,B$と整数$K$が与えられる
- $A,B$の長さ$k(\geq K)$の部分文字列のペア$A[i\dots(i+k)],B[j\dots(j+k)]$のうち、先頭$K$文字が一致するものの個数を数え上げよ
### 制約
- $1\leq |A|,|B|\leq 10^5$
- $1\leq K\leq \min\{|A|,|B|\}$
## 解法メモ
- LCP配列を用いて数え上げる。
- $A,B$を連結した文字列のSA,LCPを求めてうまくやろうとすると、SA上である値が$A$のものなのか$B$のものなのかを区別する必要があり、かなり面倒。


- そこで、$A,B$の区別なく数え上げた後、余計な部分を引くことを考える。


- $F(S)$を、文字列$S$の異なる位置にある部分文字列のペアのうち、先頭$K$文字が一致するものの個数 とする
- これは [[Codeforces 123D String]] と似たような問題になり、LCP配列の区間の内、最小値が$K$以下になるものについて、$\min - K + 1$の合計を求めることで求めることができる
	- これはstackを使って、ヒストグラム内最大長方形と似たようなコードで線形時間で解くことができる。


- これを使えば、最終的に求めたいものは$F(A+B)-F(A)-F(B)$となる。
## 実装例
最初setをつかって$O((|A|+|B|)\log (|A|+|B|))$で書いたがTLが厳しくて通らず、stackを使った線形時間の解法で通した。
```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <set>
#include <stack>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define rep2(i, m, n) for (int i = (m), i##_len = (n); i < i##_len; ++i)
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

// ヒストグラムの中で、h[i]が最小となるような区間を返す。ただし、h[i]が一致する場合はiが小さいほうが小さいと見なす
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
ll range_min_sum_leq_k_(const vector<int>& v, int k) {
    const int n = v.size();
    vector<pair<int, int> > lr = largest_rectangle_in_histogram<int>(v);
    ll res = 0;
    rep(i, n) {
        if (v[i] >= k) {
            ll ln = i - lr[i].first + 1, rn = lr[i].second - i + 1;
            res += ln * rn * (v[i] - k + 1);
        }
    }
    return res;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int k;
        cin >> k;
        if (k == 0) break;
        string a, b;
        cin >> a >> b;
        int n = a.size(), m = b.size();

        string ab = a;
        ab.push_back('{');
        rep(i, m) ab.push_back(b[i]);

        vector<int> sa_ab = sa_is(ab);
        vector<int> la_ab = lcp(ab, sa_ab);

        vector<int> sa_a = sa_is(a);
        vector<int> la_a = lcp(a, sa_a);
        vector<int> sa_b = sa_is(b);
        vector<int> la_b = lcp(b, sa_b);

        ll ans = 0;
        ans += range_min_sum_leq_k_(la_ab, k);
        ans -= range_min_sum_leq_k_(la_a, k);
        ans -= range_min_sum_leq_k_(la_b, k);
        cout << ans << "\n";
    }
}
```