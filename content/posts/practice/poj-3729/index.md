+++
title = "POJ 3729 Facer's string"
date = 2025-03-04T13:54:21+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3729

https://vjudge.net/problem/POJ-3729
<!--more-->
## 問題概要
- 長さ$N,M$の非負整数列$S_1,S_2$と整数$K$が与えられる

- $S_1$の末尾に$S_1,S_2$のどの要素とも一致しない番兵$-1$を追加した数列を$S_1'$とする
- $S_1'$の長さ$K+1$の連続部分列$S_3$のうち以下の2条件を満たすものはいくつあるか
	1. $S_3$は$S_2$に含まれない
	2. 先頭$K$要素$S_3[1\dots K]$は$S_2$に含まれる
### 制約
- $S_1, S_2$は$10^4$以下の非負整数からなる
- $0\leq N, M\leq 5\times 10^4$
## 解法メモ
思いついた解法はSA+LCPを使う解法、ロリハを使う解法の2つ
### SA+LCPを使う解法
- $S_1'+S_2$のSA、LCPを求めてこれについて考える

- 答えは、「（長さ$K$の部分列$S_3$で先頭$K$が$S_2$に含まれるものの個数） - （長さ$K+1$の部分列$S_4$で先頭$K$が$S_2$に含まれるものの個数）」とみなすことができる
- 同じ処理を2回、文字列長$K, K+1$に対して行ってからその差を求めるだけでいい

- LCPのminが$K$以上になる区間は線形時間で列挙でき、その区間は先頭$K$文字が一致する接尾辞の集合と考えることができる。
- この区間ごとに、$S_1'$に属する接尾辞と$S_2$に属する接尾辞の数を数え上げ、後者が1つでも存在すれば前者を足し合わせていけばいい。
### ローリングハッシュを使う解法
- $S_2$について、長さ$K$の部分文字列と$K+1$の部分文字列のハッシュをset1, set2に保管しておく
- $S_1'$について各始点からの長さ$K$の部分文字列がset1に存在し、$K+1$の部分文字列がset2に存在しなければ答えに1を加算していけばよい

- 実装時にはTLがきついのでsetではなくvectorをソートしてlower_boundで存在判定を行った
## 実装例
SA解
```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
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
vector<int> sa_is(vector<int> v) {
    vector<int> cv = v;
    sort(cv.begin(), cv.end());
    cv.erase(unique(cv.begin(), cv.end()), cv.end());
    for (int i = 0; i < (int)v.size(); ++i) v[i] = lower_bound(cv.begin(), cv.end(), v[i]) - cv.begin() + 1;
    return sa_is(v, cv.size());
}
vector<int> lcp(const vector<int>& s, const vector<int>& sa) {
    assert(sa.size() == s.size());
    const int n = s.size();
    vector<int> rank(n), _lcp(n - 1);
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
    int n, m, k;
    while (cin >> n >> m >> k) {
        vector<int> s1(n), s2(m);
        rep(i, n) cin >> s1[i];
        rep(i, m) cin >> s2[i];

        vector<int> s1s2 = s1;
        s1s2.push_back(1000000000);
        rep(i, m) s1s2.push_back(s2[i]);

        vector<int> sa = sa_is(s1s2);
        vector<int> lcp_ = lcp(s1s2, sa);

        ll cnt_k = 0, cnt_k_1 = 0;
        rep(i, n + m) {
            if (lcp_[i] < k) continue;
            int j = i;
            ll cnt_s1 = sa[i] < n, cnt_s2 = 1 - cnt_s1;
            while (j < n + m && lcp_[j] >= k) {
                if (sa[j + 1] < n) {
                    cnt_s1++;
                } else {
                    cnt_s2++;
                }
                j++;
            }
            if (cnt_s2 > 0) cnt_k += cnt_s1;
            i = j - 1;
        }
        rep(i, n + m) {
            if (lcp_[i] < k + 1) continue;
            int j = i;
            ll cnt_s1 = sa[i] < n, cnt_s2 = 1 - cnt_s1;
            while (j < n + m && lcp_[j] >= k + 1) {
                if (sa[j + 1] < n) {
                    cnt_s1++;
                } else {
                    cnt_s2++;
                }
                j++;
            }
            if (cnt_s2 > 0) cnt_k_1 += cnt_s1;
            i = j - 1;
        }
        cout << cnt_k - cnt_k_1 << '\n';
    }
}
```


ロリハ解
```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <ctime>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

ll MOD[] = {
    999999503, 999999527, 999999541, 999999587, 999999599, 999999607, 999999613, 999999667, 999999677, 999999733, 999999739, 999999751, 999999757, 999999761, 999999797, 999999883, 999999893, 999999929, 999999937, 1000000007, 1000000009, 1000000021, 1000000033, 1000000087, 1000000093, 1000000097, 1000000103, 1000000123, 1000000181, 1000000207, 1000000223, 1000000241, 1000000271, 1000000289, 1000000297, 1000000321, 1000000349, 1000000363, 1000000403, 1000000409, 1000000411, 1000000427, 1000000433, 1000000439, 1000000447, 1000000453, 1000000459, 1000000483,
};
class RollingHash {
   private:
    ll MOD1, MOD2, BASE1, BASE2;

    vector<ll> power1, power2;

   public:
    struct Hash {
        ll h1, h2;
        int length;
        Hash(ll h1, ll h2, int length) : h1(h1), h2(h2), length(length) {}
        bool operator<(const Hash& rhs) const {
            if (length != rhs.length) return length < rhs.length;
            if (h1 != rhs.h1) return h1 < rhs.h1;
            return h2 < rhs.h2;
        }
        bool operator==(const Hash& rhs) const { return h1 == rhs.h1 && h2 == rhs.h2 && length == rhs.length; }
    };
    typedef pair<vector<ll>, vector<ll> > Hashes;
    RollingHash() {
        srand(time(0));
        MOD1 = MOD[rand() % 50];
        do {
            MOD2 = MOD[rand() % 50];
        } while (MOD1 == MOD2);
        BASE1 = rand() % 1000 + 1000;
        do {
            BASE2 = rand() % 1000 + 1000;
        } while (BASE1 == BASE2);
        power1.push_back(1);
        power2.push_back(1);
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
        return make_pair(hash1, hash2);
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
        return make_pair(hash1, hash2);
    }
    Hash get(const Hashes& hashes, int l, int r) {
        assert(0 <= l && l <= r && r <= hashes.first.size() - 1);
        return Hash(((hashes.first[r] - hashes.first[l] * power1[r - l]) % MOD1 + MOD1) % MOD1, ((hashes.second[r] - hashes.second[l] * power2[r - l]) % MOD2 + MOD2) % MOD2, r - l);
    }
    Hash concat(const Hash& h1, const Hash& h2) {
        ll h1_1 = h1.h1, h1_2 = h1.h2, h2_1 = h2.h1, h2_2 = h2.h2;
        ll h1_length = h1.length, h2_length = h2.length;
        return Hash((h1_1 * power1[h2_length] + h2_1) % MOD1, (h1_2 * power2[h2_length] + h2_2) % MOD2, h1_length + h2_length);
    }
} RH;
typedef RollingHash::Hashes Hashes;
typedef RollingHash::Hash Hash;
// Usage: string s = "abc"; Hashes rh = RH(s); Hash hash = RH.get(rh, 0, 3);

int main() {
    int n, m, k;
    while (scanf("%d%d%d", &n, &m, &k) != EOF) {
        vector<int> s1(n), s2(m);
        rep(i, n) scanf("%d", &s1[i]);
        rep(i, m) scanf("%d", &s2[i]);

        s1.push_back(10000000);
        Hashes rh1 = RH(s1);
        Hashes rh2 = RH(s2);

        vector<Hash> st1, st2;
        rep(i, m - k + 1) { st1.push_back(RH.get(rh2, i, i + k)); }
        rep(i, m - (k + 1) + 1) { st2.push_back(RH.get(rh2, i, i + (k + 1))); }
        sort(st1.begin(), st1.end());
        sort(st2.begin(), st2.end());

        ll ans = 0;
        rep(i, (n + 1) - (k + 1) + 1) {
            Hash h1 = RH.get(rh1, i, i + k);
            Hash h2 = RH.get(rh1, i, i + (k + 1));
            vector<Hash>::iterator it1 = lower_bound(st1.begin(), st1.end(), h1);
            vector<Hash>::iterator it2 = lower_bound(st2.begin(), st2.end(), h2);
            bool f1 = it1 != st1.end() && *it1 == h1;
            bool f2 = it2 != st2.end() && *it2 == h2;
            if (f1 && !f2) ans++;
        }
        printf("%lld\n", ans);
    }
}
```
