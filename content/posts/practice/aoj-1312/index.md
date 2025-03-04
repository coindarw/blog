+++
title = "AOJ 1312 Where's Wally"
date = 2025-03-04T13:52:55+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1312

https://vjudge.net/problem/Aizu-1312
<!--more-->
## 問題概要
- base64形式で$H\times W$の2値画像、$P\times P$の2値パターンが与えられる
- 画像内の$P\times P$領域の中で、パターンの0・90・180・270度回転・反転いずれかに一致するものはいくつあるか
### 制約
- $1\leq W\leq 1000$
- $1\leq H\leq 1000$
- $1\leq P\leq 1000$

## 解法メモ
- 2次元でRolling Hashをやればよい

- 前処理として、あらかじめ画像とパターン（反転・回転）のハッシュを計算しておく
- 画像の中の各$P\times P$領域のハッシュがパターンの反転・回転8通りのいずれかと一致する場所をカウントしていけばよい。
- ~~（書いていて思ったが8種とハッシュをかけている...？）~~
## 実装例
BASE64の中身を初めて調べた
Rolling Hashに加えてBASE64デコード処理や回転・反転も書かないといけなくてまあまあ実装が面倒
```cpp
#include <algorithm>
#include <array>
#include <cassert>
#include <iostream>
#include <random>
#include <set>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

vector<bool> base64_decoding(const string& s) {
    assert(s.size() % 4 == 0);
    vector<bool> res(s.size() * 6);
    for (size_t i = 0; i < s.size(); i += 4) {
        unsigned x = 0;
        for (size_t j = 0; j < 4; ++j) {
            x <<= 6;
            if (s[i + j] == '=') x |= 0;
            else if ('A' <= s[i + j] && s[i + j] <= 'Z') x |= s[i + j] - 'A' + 0;
            else if ('a' <= s[i + j] && s[i + j] <= 'z') x |= s[i + j] - 'a' + 26;
            else if ('0' <= s[i + j] && s[i + j] <= '9') x |= s[i + j] - '0' + 52;
            else if (s[i + j] == '+') x |= 62;
            else if (s[i + j] == '/') x |= 63;
            else assert(false);
        }
        for (size_t j = 0; j < 24; ++j) res[i * 6 + j] = x & (1u << (23 - j));
    }
    return res;
}

template <size_t MOD_NUM>
struct RollingHash2d {
    using Hash = array<int, MOD_NUM>;
    using Hashes = vector<vector<Hash>>;
    static_assert(MOD_NUM < 40, "MOD_NUM must be less than 40");
    array<long long, 50> MODS = {
        999999503, 999999527, 999999541, 999999587, 999999599, 999999607, 999999613, 999999667, 999999677, 999999733, 999999739, 999999751, 999999757, 999999761, 999999797, 999999883, 999999893, 999999929, 999999937, 1000000007, 1000000009, 1000000021, 1000000033, 1000000087, 1000000093, 1000000097, 1000000103, 1000000123, 1000000181, 1000000207, 1000000223, 1000000241, 1000000271, 1000000289, 1000000297, 1000000321, 1000000349, 1000000363, 1000000403, 1000000409, 1000000411, 1000000427, 1000000433, 1000000439, 1000000447, 1000000453, 1000000459, 1000000483, 1000000513, 1000000531,
    };

   private:
    array<long long, MOD_NUM> MOD;
    array<long long, MOD_NUM * 2> BASE;
    vector<vector<long long>> power_table, power_table_inv;

    long long mod_inv(long long a, long long M) {
        long long b = M, u = 1, v = 0;
        while (b) {
            long long t = a / b;
            a -= t * b;
            swap(a, b);
            u -= t * v;
            swap(u, v);
        }
        u %= M;
        if (u < 0) u += M;
        return u;
    }

    long long power(size_t x, size_t baseIdx) {
        while (power_table[baseIdx].size() <= x) {
            power_table[baseIdx].push_back((power_table[baseIdx].back() * BASE[baseIdx] % MOD[baseIdx / 2]));
            power_table_inv[baseIdx].push_back(mod_inv(power_table[baseIdx].back(), MOD[baseIdx / 2]));
        }
        return power_table[baseIdx][x];
    }
    long long power_inv(size_t x, size_t baseIdx) {
        while (power_table_inv[baseIdx].size() <= x) {
            power_table[baseIdx].push_back((power_table[baseIdx].back() * BASE[baseIdx] % MOD[baseIdx / 2]));
            power_table_inv[baseIdx].push_back(mod_inv(power_table[baseIdx].back(), MOD[baseIdx / 2]));
        }
        return power_table_inv[baseIdx][x];
    }

    template <typename T>
    vector<vector<int>> _compute_hash(const vector<vector<T>>& a, size_t mIdx) {
        const size_t n = a.size();
        const size_t m = a[0].size();
        vector<vector<int>> hash_(n + 1, vector<int>(m + 1, 0));
        size_t b1Idx = mIdx * 2, b2Idx = mIdx * 2 + 1;
        long long mod = MOD[mIdx];

        for (size_t i = 1; i <= n; ++i)
            for (size_t j = 1; j <= m; ++j) hash_[i][j] = a.at(i - 1).at(j - 1) % mod;
        for (size_t i = 1; i <= n; ++i)
            for (size_t j = 1; j <= m; ++j) {
                hash_[i][j] = (hash_[i][j] * power(j - 1, b1Idx)) % mod;
                hash_[i][j] += hash_[i][j - 1];
                if (hash_[i][j] >= mod) hash_[i][j] -= mod;
            }
        for (size_t i = 1; i <= n; ++i)
            for (size_t j = 1; j <= m; ++j) {
                hash_[i][j] = (hash_[i][j] * power(i - 1, b2Idx)) % mod;
                hash_[i][j] += hash_[i - 1][j];
                if (hash_[i][j] >= mod) hash_[i][j] -= mod;
            }
        return hash_;
    }

    int _get_hash(const Hashes& hashes, size_t i, size_t j, size_t h, size_t w, size_t mIdx) {
        size_t b1Idx = mIdx * 2, b2Idx = mIdx * 2 + 1;
        int res = hashes[i + h - 1][j + w - 1][mIdx];
        res -= hashes[i - 1][j + w - 1][mIdx];
        if (res < 0) res += MOD[mIdx];
        res -= hashes[i + h - 1][j - 1][mIdx];
        if (res < 0) res += MOD[mIdx];
        res += hashes[i - 1][j - 1][mIdx];
        if (res >= MOD[mIdx]) res -= MOD[mIdx];
        res = (res * power_inv(j - 1, b1Idx)) % MOD[mIdx];
        res = (res * power_inv(i - 1, b2Idx)) % MOD[mIdx];
        return res;
    }

   public:
    RollingHash2d() {
        random_device rnd;
        set<int> mods_idx;
        while (mods_idx.size() < size_t(MOD_NUM)) mods_idx.insert(rnd() % MODS.size());
        vector<int> tmp(mods_idx.begin(), mods_idx.end());
        for (size_t i = 0; i < MOD_NUM; ++i) MOD[i] = MODS[tmp[i]];

        for (size_t i = 0; i < MOD_NUM * 2; ++i) BASE[i] = rnd() % (MOD[i / 2] - 2) + 2;
        power_table.resize(MOD_NUM * 2, vector<long long>(1, 1));
        power_table_inv.resize(MOD_NUM * 2, vector<long long>(1, 1));
    }

    template <typename T>
    Hashes compute_hash(const vector<vector<T>>& a) {
        const size_t n = a.size();
        assert(n >= 0);
        const size_t m = a[0].size();
        assert(m >= 0);
        Hashes res(n + 1, vector<Hash>(m + 1));
        for (size_t i = 0; i < MOD_NUM; ++i) {
            vector<vector<int>> hashes = _compute_hash(a, i);
            for (size_t y = 1; y <= n; ++y)
                for (size_t x = 1; x <= m; ++x) {
                    res[y][x][i] = hashes[y][x];
                }
        }
        return res;
    }
    Hashes compute_hash(const vector<string>& a) {
        const int n = int(a.size()) - 1;
        assert(n >= 1);
        const int m = int(a[1].size()) - 1;
        assert(m >= 1);
        vector<vector<int>> a_(n + 1, vector<int>(m + 1));
        for (size_t i = 1; i <= n; ++i)
            for (size_t j = 1; j <= m; ++j) a_[i][j] = a[i][j];
        return compute_hash(a_);
    }

    Hash get_hash(const Hashes& hashes, size_t i, size_t j, size_t h, size_t w) {
        assert(i >= 0 && j >= 0 && i + h <= hashes.size() && j + w <= hashes[0].size());
        ++i, ++j;
        Hash res;
        for (size_t k = 0; k < MOD_NUM; ++k) res[k] = _get_hash(hashes, i, j, h, w, k);
        return res;
    }
};
using RollingHash = RollingHash2d<2>;
RollingHash RH;

template <typename T>
void rotate_invert(vector<vector<T>>& a, int rotate, bool invert) {
    const int n = a.size();
    const int m = a[0].size();
    assert(n == m);
    assert(0 <= rotate && rotate < 4);
    vector<vector<T>> b = a;
    if (rotate == 1) {
        rep(i, n) rep(j, m) a[i][j] = b[j][n - i - 1];
    } else if (rotate == 2) {
        rep(i, n) rep(j, m) a[i][j] = b[n - i - 1][m - j - 1];
    } else if (rotate == 3) {
        rep(i, n) rep(j, m) a[i][j] = b[m - j - 1][i];
    }
    if (invert) {
        rep(i, n) reverse(a[i].begin(), a[i].end());
    }
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    while (true) {
        int w, h, p;
        cin >> w >> h >> p;
        if (w == 0 && h == 0 && p == 0) break;
        vector<vector<bool>> image(h), pattern(p);
        rep(i, h) {
            string s;
            cin >> s;
            while (s.size() % 4 != 0) s.push_back('=');  // この問題では必要ないが、base64の仕様に合わせている
            image[i] = base64_decoding(s);
            image[i].resize(w);
        }
        rep(i, p) {
            string s;
            cin >> s;
            while (s.size() % 4 != 0) s.push_back('=');
            pattern[i] = base64_decoding(s);
            pattern[i].resize(p);
        }

        auto image_hashes = RH.compute_hash(image);

        vector<RollingHash::Hash> pattern_hash(8);  // 回転4通り * 反転2通りで8通りのパターンのハッシュを計算
        rep(i, 4) {
            rep(j, 2) {
                auto pattern_ = pattern;
                rotate_invert(pattern_, i, j);
                auto pattern_hashes_ = RH.compute_hash(pattern_);
                pattern_hash[i * 2 + j] = RH.get_hash(pattern_hashes_, 0, 0, p, p);
            }
        }

        int ans = 0;
        rep(i, h - p + 1) {
            rep(j, w - p + 1) {
                auto image_hash = RH.get_hash(image_hashes, i, j, p, p);
                bool found = false;
                rep(k, 8) {
                    if (image_hash == pattern_hash[k]) {
                        found = true;
                        break;
                    }
                }
                if (found) ++ans;
            }
        }

        cout << ans << "\n";
    }
}
```