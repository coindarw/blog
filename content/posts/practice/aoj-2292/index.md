+++
title = "AOJ 2292 Common Palindromes(BOJ 22496)"
date = 2025-03-04T13:53:01+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2292
https://www.acmicpc.net/problem/22496

https://vjudge.net/problem/Aizu-2292
https://vjudge.net/problem/Baekjoon-22496

なお、yukicoderには$|S|,|T|$の制約が10倍になったこと以外は同じ問題「Common Palindromes Extra」が存在する
https://yukicoder.me/problems/no/263
https://vjudge.net/problem/Yukicoder-263
<!--more-->
## 問題概要
- 文字列$S,T$が与えられる。$S,T$の部分文字列の組$(s,t)$で、$s=t$かつそれらが回文であるものの個数を数えよ
- 部分文字列は場所が異なれば違うとみなす
### 制約
- $1\leq |S|,|T|\leq 5\times 10^4$
- $S,T$は英大文字からなる

## 解法メモ
- $S$に登場する回文の種類数を$N$とすると$N\leq |S|$になる
	- ある文字列の末尾に1文字追加しても回文が2種類増えることはないから
	- （あるABC-Exの解説に載っていたので知った。ネタバレ防止 https://kmyk.github.io/cp-unspoiler/?q=aHR0cHM6Ly9hdGNvZGVyLmpwL2NvbnRlc3RzL2FiYzIzNy90YXNrcy9hYmMyMzdfaA%3D%3D ）

- 回文ごとにLCP等を使って数えていけばよさそう

- 回文の列挙方法を知らないので考えてみる
	- manacherのアルゴリズムを使えば各場所を中心とする最長回文を$O(|S|)$で列挙できる（数えられるのは奇数長の回文なので、`A$B$C$D` のような感じで間に使わない文字を挟む）
	- 最長回文長$m[i]$を$i$昇順に見ていく
		- $i$を中心とする半径$m[i], m[i]-1, \cdots, 1$の回文を見る
			- 既に列挙済みであれば次の場所に行く
			- そうでなければ列挙する
- というようにすれば列挙できる。既に列挙済みかどうかチェックする方法は以下のように適当に
	- ハッシュをsetに入れておく
	- SAで左端のindexが出てくる場所をsetに入れておき、左右の要素とのLCPが回文の直径以上か見る


- このような方法で回文の区間を列挙しておいたあと、
	- $S+T$ののLCPを見て、同じ回文のペアの個数を数える(1)
	- $S$のLCPを見て、同じ回文のペアの個数を数える(2)
	- $T$のLCPを見て、同じ回文のペアの個数を数える(3)
	- (1)-(2)-(3)を求める
- で答えが求められる
- `A` 1文字や `B` 1文字は回文だが、manacherの時に追加した `$` 1文字は回文にならないのでその分は引く必要がある

- [2011/Practice/夏合宿/講評 - ICPC OB/OG の会 (jag-icpc.org)](https://jag-icpc.org/?2011%2FPractice%2F%E5%A4%8F%E5%90%88%E5%AE%BF%2F%E8%AC%9B%E8%A9%95) に解説があり、想定解は少し違う
- 他の人の実装も見た感じPalindromic Treeというのを使うと回文列挙を線形時間でできるらしい
	- LCに入っていた https://judge.yosupo.jp/problem/eertree
## 実装例
AOJではこのLCP+SparseTable上二分探索したコードで通るが、yukicoderのExtraの方はTLEしてしまった
Extraの方で通るコードは記事末尾
```cpp
#include <algorithm>
#include <cassert>
#include <cmath>
#include <iostream>
#include <set>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
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

vector<int> manacher(const string& s) {
    const int n = s.size();
    int i = 0, j = 0;
    vector<int> res(n);
    while (i < n) {
        while (i - j >= 0 && i + j < n && s[i - j] == s[i + j]) ++j;
        res[i] = j;
        int k = 1;
        while (i - k >= 0 && k + res[i - k] < j) res[i + k] = res[i - k], ++k;
        i += k, j -= k;
    }
    return res;
}

template <typename T, T INF>
class SparseTableMin {
    vector<vector<T>> st;
    vector<int> lookup;
    int n, N, logN;

    int __bit_ceil(int num) { return 1 << (32 - __builtin_clz(num)); }

   public:
    SparseTableMin() {}
    SparseTableMin(const vector<T>& v) {
        n = v.size();
        if (n == 0) return;
        N = __bit_ceil(v.size());
        logN = 32 - __builtin_clz(N);
        st.assign(logN, vector<T>(N, INF));
        for (int i = 0; i < n; i++) st[0][i] = v[i];
        int len = 1;
        for (int i = 1; i < (int)st.size(); i++) {
            for (int j = 0; j + len < N; j++) {
                st[i][j] = min(st[i - 1][j], st[i - 1][j + len]);
            }
            len <<= 1;
        }
        lookup.resize(N + 1);
        for (int i = 2; i < (int)lookup.size(); i++) lookup[i] = lookup[i >> 1] + 1;
    }

    T prod(int l, int r) {
        assert(0 <= l && l <= r && r <= n);
        if (l == r) return INF;
        int b = lookup[r - l];
        return min(st[b][l], st[b][r - (1 << b)]);
    }
};

vector<pair<int, int>> enumerate_palindromes(const string& s, const vector<int>& sa, const vector<int>& la) {
    const int n = s.size();
    vector<pair<int, int>> res;
    string t(n * 2 - 1, '$');
    for (int i = 0; i < n; ++i) t[i * 2] = s[i];

    SparseTableMin<int, inf> st(la);
    vector<int> rev_sa(n);
    for (int i = 0; i < n; ++i) rev_sa[sa[i]] = i;

    vector<int> man = manacher(t);
    vector<set<int>> lefts(n + 1);

    for (int i = 0; i < n * 2 - 1; ++i) {
        for (int rad = man[i]; rad > 0; --rad) {
            int l = i - (rad - 1);
            int r = i + (rad - 1);
            if (l % 2 != 0) continue;
            l = l / 2;
            r = r / 2 + 1;
            int len = r - l;
            auto ub = lefts[len].upper_bound(rev_sa[l]);
            if (ub != lefts[len].end()) {
                if (st.prod(min(rev_sa[l], *ub), max(rev_sa[l], *ub)) >= len) {
                    break;
                }
            }
            if (ub != lefts[len].begin()) {
                ub--;
                if (st.prod(min(rev_sa[l], *ub), max(rev_sa[l], *ub)) >= len) {
                    break;
                }
            }
            lefts[len].insert(rev_sa[l]);
            res.emplace_back(l, r);
        }
    }

    return res;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    string s, t;
    cin >> s >> t;

    auto count_palindrome_pairs = [](string str) -> ll {
        auto sa = sa_is(str);
        auto la = lcp(str, sa);
        auto pal = enumerate_palindromes(str, sa, la);
        vector<int> rev(str.size());
        rep(i, str.size()) rev[sa[i]] = i;
        SparseTableMin<int, inf> st(la);

        ll res = 0;
        for (auto [l, r] : pal) {
            auto bsearch = [&](auto cmp, int ok, int ng) {
                while (abs(ng - ok) > 1) {
                    int mid = (ng + ok) / 2;
                    if (cmp(mid)) ok = mid;
                    else ng = mid;
                }
                return ok;
            };
            // [l, r)とのLCPがr-l以上のものの個数を数える
            ll ls = bsearch([&](int x) { return st.prod(min(rev[l], rev[l] - x), max(rev[l], rev[l] - x)) >= r - l; }, 0, 1 + rev[l]);
            ll rs = bsearch([&](int x) { return st.prod(min(rev[l], rev[l] + x), max(rev[l], rev[l] + x)) >= r - l; }, 0, 1 + int(str.size()) - 1 - rev[l]);
            ll cnt = rs - (-ls) + 1;
            res += cnt * (cnt - 1) / 2;
        }
        return res;
    };

    ll ans = 0;
    ans += count_palindrome_pairs(s + "/@" + t);  // 区切り文字を中心とする回文ができないように適当な記号2つで区切る
    ans -= count_palindrome_pairs(s);
    ans -= count_palindrome_pairs(t);

    cout << ans << '\n';
}
```


回文列挙をRollingHashとunordered_setで線形時間で行えるようにし、ペアの個数をカウントするときはatcoder::segtree.max_right, min_leftを使うようにしたら1ケースだけ通らず

unordered_setをgp_hash_tableに変え、いくつかの#pragmaディレクティブ(いわゆるQCFium法)を付けたら通った

想定解法が回文木なので余計なlogのつくこの解法だとかなりTLがきつい
```cpp
#pragma GCC target("avx2")
#pragma GCC optimize("O3")
#pragma GCC optimize("unroll-loops")
#include <algorithm>
#include <atcoder/segtree>
#include <cassert>
#include <ext/pb_ds/assoc_container.hpp>
#include <iostream>
#include <random>
#include <unordered_set>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
using namespace std;
using namespace __gnu_pbds;

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

vector<int> manacher(const string& s) {
    const int n = s.size();
    int i = 0, j = 0;
    vector<int> res(n);
    while (i < n) {
        while (i - j >= 0 && i + j < n && s[i - j] == s[i + j]) ++j;
        res[i] = j;
        int k = 1;
        while (i - k >= 0 && k + res[i - k] < j) res[i + k] = res[i - k], ++k;
        i += k, j -= k;
    }
    return res;
}

template <typename T, T INF>
class SparseTableMin {
    vector<vector<T>> st;
    vector<int> lookup;
    int n, N, logN;

    int __bit_ceil(int num) { return 1 << (32 - __builtin_clz(num)); }

   public:
    SparseTableMin() {}
    SparseTableMin(const vector<T>& v) {
        n = v.size();
        if (n == 0) return;
        N = __bit_ceil(v.size());
        logN = 32 - __builtin_clz(N);
        st.assign(logN, vector<T>(N, INF));
        for (int i = 0; i < n; i++) st[0][i] = v[i];
        int len = 1;
        for (int i = 1; i < (int)st.size(); i++) {
            for (int j = 0; j + len < N; j++) {
                st[i][j] = min(st[i - 1][j], st[i - 1][j + len]);
            }
            len <<= 1;
        }
        lookup.resize(N + 1);
        for (int i = 2; i < (int)lookup.size(); i++) lookup[i] = lookup[i >> 1] + 1;
    }

    T prod(int l, int r) {
        assert(0 <= l && l <= r && r <= n);
        if (l == r) return INF;
        int b = lookup[r - l];
        return min(st[b][l], st[b][r - (1 << b)]);
    }
};

class RollingHash {
   private:
    vector<ll> MOD = {
        999999503, 999999527, 999999541, 999999587, 999999599, 999999607, 999999613, 999999667, 999999677, 999999733, 999999739, 999999751, 999999757, 999999761, 999999797, 999999883, 999999893, 999999929, 999999937, 1000000007, 1000000009, 1000000021, 1000000033, 1000000087, 1000000093, 1000000097, 1000000103, 1000000123, 1000000181, 1000000207, 1000000223, 1000000241, 1000000271, 1000000289, 1000000297, 1000000321, 1000000349, 1000000363, 1000000403, 1000000409, 1000000411, 1000000427, 1000000433, 1000000439, 1000000447, 1000000453, 1000000459, 1000000483,
    };
    vector<ll> BASE = {
        999611, 999613, 999623, 999631, 999653, 999667, 999671, 999683, 999721, 999727, 999749, 999763, 999769, 999773, 999809, 999853, 999863, 999883, 999907, 999917, 999931, 999953, 999959, 999961, 999979, 999983, 1000003, 1000033, 1000037, 1000039, 1000081, 1000099, 1000117, 1000121, 1000133, 1000151, 1000159, 1000171, 1000183, 1000187, 1000193, 1000199, 1000211, 1000213, 1000231, 1000249, 1000253, 1000273, 1000289, 1000291, 1000303, 1000313, 1000333, 1000357, 1000367, 1000381, 1000393, 1000397, 1000403,
    };
    ll MOD1, MOD2, BASE1, BASE2;

    vector<ll> power1, power2;

   public:
    using Hash = tuple<ll, ll, ll>;
    using Hashes = pair<vector<ll>, vector<ll>>;
    RollingHash() {
        random_device rnd;
        MOD1 = MOD[rnd() % MOD.size()];
        do {
            MOD2 = MOD[rnd() % MOD.size()];
        } while (MOD1 == MOD2);
        BASE1 = BASE[rnd() % BASE.size()];
        do {
            BASE2 = BASE[rnd() % BASE.size()];
        } while (BASE1 == BASE2);
        power1 = {1}, power2 = {1};
    }

    Hashes operator()(const string& str) {
        int size = str.size();
        vector<ll> hash1(size + 1), hash2(size + 1);
        for (int i = 1; i <= size; i++) {
            hash1[i] = (hash1[i - 1] + str[i - 1]) * BASE1 % MOD1;
            hash2[i] = (hash2[i - 1] + str[i - 1]) * BASE2 % MOD2;
        }
        while (power1.size() <= size) {
            power1.push_back(power1.back() * BASE1 % MOD1);
            power2.push_back(power2.back() * BASE2 % MOD2);
        }
        return {hash1, hash2};
    }
    template <typename T>
    Hashes operator()(const vector<T>& v) {
        int size = v.size();
        vector<ll> hash1(size + 1), hash2(size + 1);
        for (int i = 1; i <= size; i++) {
            hash1[i] = (hash1[i - 1] + v[i - 1] % MOD1) * BASE1 % MOD1;
            hash2[i] = (hash2[i - 1] + v[i - 1] % MOD2) * BASE2 % MOD2;
        }
        while (power1.size() <= size) {
            power1.push_back(power1.back() * BASE1 % MOD1);
            power2.push_back(power2.back() * BASE2 % MOD2);
        }
        return {hash1, hash2};
    }
    Hash get(const Hashes& hashes, int l, int r) {
        assert(0 <= l && l <= r && r <= hashes.first.size() - 1);
        return make_tuple(((hashes.first[r] - hashes.first[l] * power1[r - l]) % MOD1 + MOD1) % MOD1, ((hashes.second[r] - hashes.second[l] * power2[r - l]) % MOD2 + MOD2) % MOD2, r - l);
    }
    Hash concat(const Hash& h1, const Hash& h2) {
        auto [h1_1, h1_2, h1_length] = h1;
        auto [h2_1, h2_2, h2_length] = h2;
        return make_tuple((h1_1 * power1[h2_length] + h2_1) % MOD1, (h1_2 * power2[h2_length] + h2_2) % MOD2, h1_length + h2_length);
    }
} RH;
using Hashes = RollingHash::Hashes;
using Hash = RollingHash::Hash;
Hash operator+(const Hash& a, const Hash& b) noexcept { return RH.concat(a, b); }
// Usage: string s = "abc"; Hashes rh = RH(s); Hash hash = RH.get(rh, 0, 3);

vector<pair<int, int>> enumerate_palindromes(const string& s) {
    const int n = s.size();
    vector<pair<int, int>> res;
    string t(n * 2 - 1, '$');
    for (int i = 0; i < n; ++i) t[i * 2] = s[i];

    Hashes rh = RH(s);
    gp_hash_table<uint64_t, int> table;

    vector<int> man = manacher(t);

    for (int i = 0; i < n * 2 - 1; ++i) {
        for (int rad = man[i]; rad > 0; --rad) {
            int l = i - (rad - 1);
            int r = i + (rad - 1);
            if (l % 2 != 0) continue;
            l = l / 2;
            r = r / 2 + 1;
            int len = r - l;
            Hash hash = RH.get(rh, l, r);
            uint64_t h = get<0>(hash) * 1e9 + get<1>(hash) + get<2>(hash);
            if (table.find(h) != table.end()) break;
            table[h] = 1;
            res.emplace_back(l, r);
        }
    }

    return res;
}

int op(int a, int b) { return min(a, b); }
int e() { return inf; }
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    string s, t;
    cin >> s >> t;

    auto count_palindrome_pairs = [](string str) -> ll {
        auto sa = sa_is(str);
        auto la = lcp(str, sa);
        auto pal = enumerate_palindromes(str);
        vector<int> rev(str.size());
        rep(i, str.size()) rev[sa[i]] = i;
        atcoder::segtree<int, op, e> st(la);

        ll res = 0;
        for (auto [l, r] : pal) {
            int ls = st.min_left(rev[l], [&](int x) { return x >= r - l; });
            int rs = st.max_right(rev[l], [&](int x) { return x >= r - l; });
            ll cnt = rs - ls + 1;
            res += cnt * (cnt - 1) / 2;
        }

        return res;
    };

    ll ans = 0;
    ans += count_palindrome_pairs(s + "/@" + t);  // 区切り文字を中心とする回文ができないように適当な記号2つで区切る
    ans -= count_palindrome_pairs(s);
    ans -= count_palindrome_pairs(t);

    cout << ans << '\n';
}
```