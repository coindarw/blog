+++
title = "AOJ 2214 Warp Hall"
date = 2025-03-04T13:52:59+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://onlinejudge.u-aizu.ac.jp/challenges/search/titles/2214

https://vjudge.net/problem/Aizu-2214

**EDPCの問題の若干のネタバレを含みます**
<!--more-->

## 問題概要
- 座標$(1,1)$から、上移動$(+1,0)$と横移動$(0,+1)$を繰り返して座標$(N,M)$に移動する方法の数を$\mod 10^9+7$で求めよ。
- ただし、グリッド上には$K$個のワープホールがあり、座標$(a_i,b_i)$に辿り着いた時点で$(c_i,d_i)$にワープする。ワープホールの座標は$a_i\leq c_i, b_i\leq d_i, (a_i,b_i)\neq (c_i,d_i)$を満たす。
### 制約
- $1\leq N,M \leq 10^5, 0\leq K\leq 10^3$
- $a_i\leq c_i, b_i\leq d_i, (a_i,b_i)\neq (c_i,d_i)$
## 解法メモ

- EDPC-Yに似ており、あるワープホールを通ったか・通っていないかの管理が面倒そうなので包徐原理を使って数え上げができそう

- `dp[i] = i番目のワープホールの出口まで来る方法の数` だと出口が一致する複数のワープホールも存在するので面倒そう

- `dp[i] = i番目のワープホールの入り口まで来る方法の数`　として、 `dp[i]` を求める方法を考えてみる

- これは、$(1,1)$からワープホールを無視して移動する方法$\binom{(a_i-1)+(b_i-1)}{a_i-1}$通りから、他のワープホールを通っているのに無視したinvalidな移動方法を引いた後、ワープホールを通って移動してきた移動方法を足せば求められる

- ワープホールを複数回通るようなルートを考えた時にルートが重複してしまわないか心配になるが、グリッドを書いてどのルートを足すor引くのか考えてみるとうまくいくことがわかる

## 実装例

いわゆる徐原理といわれるものだと思う

modintや二項係数が行数をかなりとっているが、実際に問題を解いているパートはかなり短い
```cpp
#include <bits/stdc++.h>

using ll = long long;
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

template <ll MOD>
class mod_int {
    ll _val;
    // +, -, +,-,*,/, ==, !=, <<, >>, ++, --, +=, -=, *=, /=, inv, pow
   public:
    mod_int(const ll x = 0) noexcept : _val(x % MOD) {
        if (_val < 0) {
            _val += MOD;
        }
    }
    ll& val() noexcept { return _val; }
    constexpr const ll& val() const noexcept { return _val; }
    mod_int operator+() const noexcept { return *this; }
    mod_int operator-() const noexcept { return mod_int(-_val); }
    mod_int& operator+=(const mod_int& r) noexcept {
        _val += r._val;
        if (_val >= MOD) _val -= MOD;
        return *this;
    }
    mod_int& operator-=(const mod_int& r) noexcept {
        if (_val < r._val) _val += MOD;
        _val -= r._val;
        return *this;
    }
    mod_int& operator*=(const mod_int& r) noexcept {
        _val = _val * r._val % MOD;
        return *this;
    }
    mod_int& operator/=(const mod_int& r) noexcept {
        ll exp = MOD - 2;
        while (exp) {
            if (exp % 2) {
                *this *= r;
            }
            r *= r;
            exp /= 2;
        }
        return *this;
    }
    bool operator==(const mod_int& r) const noexcept { return _val == r._val; }
    bool operator!=(const mod_int& r) const noexcept { return _val != r._val; }
    friend ostream& operator<<(ostream& os, const mod_int& r) noexcept { return os << r._val; }
    friend istream& operator>>(istream& is, mod_int& r) noexcept {
        ll t;
        is >> t;
        r = mod_int<MOD>(t);
        return is;
    }
    mod_int inv() const noexcept {
        mod_int res = 1;
        mod_int x = *this;
        ll n = MOD - 2;
        while (n) {
            if (n % 2) res *= x;
            x *= x;
            n /= 2;
        }
        return res;
    }
    mod_int pow(ll n) const noexcept {
        bool minus = false;
        if (n < 0) {
            minus = true;
            n = -n;
        }
        mod_int res = 1;
        mod_int x = *this;
        while (n) {
            if (n % 2) res *= x;
            x *= x;
            n /= 2;
        }
        if (minus) return res.inv();
        return res;
    }
    mod_int& operator++() noexcept {
        _val++;
        if (_val == MOD) {
            _val = 0;
        }
        return *this;
    }
    mod_int& operator--() noexcept {
        if (_val == 0) {
            _val = MOD;
        }
        _val--;
        return *this;
    }
    mod_int operator++(int) noexcept {
        mod_int res = *this;
        ++*this;
        return res;
    }
    mod_int operator--(int) noexcept {
        mod_int res = *this;
        --*this;
        return res;
    }
    friend mod_int operator+(const mod_int& lhs, const mod_int& rhs) { return mod_int(lhs) += rhs; }
    friend mod_int operator-(const mod_int& lhs, const mod_int& rhs) { return mod_int(lhs) -= rhs; }
    friend mod_int operator*(const mod_int& lhs, const mod_int& rhs) { return mod_int(lhs) *= rhs; }
    friend mod_int operator/(const mod_int& lhs, const mod_int& rhs) { return mod_int(lhs) /= rhs; }
    constexpr bool operator<(const mod_int r) const noexcept { return _val < r._val; }
    static constexpr ll mod() { return MOD; }
};

constexpr ll MOD = 1'000'000'007;
using mint = mod_int<MOD>;
struct modInv {
    int n;
    vector<mint> d;
    modInv() : n(2), d({0, 1}) {}
    mint operator()(int i) {
        while (n <= i) d.emplace_back(-d[MOD % n] * (MOD / n)), ++n;
        return d[i];
    }
    mint operator[](int i) const { return d[i]; }
} invs;
struct Factorial {
    int n;
    vector<mint> d;
    Factorial() : n(2), d({1, 1}) {}
    mint operator()(int i) {
        while (n <= i) d.emplace_back(d.back() * n), ++n;
        return d[i];
    }
    mint operator[](int i) const { return d[i]; }
} factorial;
struct FactorialInv {
    int n;
    vector<mint> d;
    FactorialInv() : n(2), d({1, 1}) {}
    mint operator()(int i) {
        while (n <= i) d.emplace_back(d.back() * invs(n)), ++n;
        return d[i];
    }
    mint operator[](int i) const { return d[i]; }
} factorialInv;
mint C(int n, int r) {
    if (n < r || n < 0 || r < 0) return 0;
    return factorial(n) * factorialInv(r) * factorialInv(n - r);
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int n, m, k;
        cin >> n >> m >> k;
        if (n == 0 && m == 0 && k == 0) break;

        using T = tuple<int, int, int, int>;
        vector<T> pos(k + 2);
        reps(i, k) {
            int a, b, c, d;
            cin >> a >> b >> c >> d;
            pos[i] = {a, b, c, d};
        }

        pos[k + 1] = {n, m, n, m};  // ゴールをk+1番目のワープホールとする

        sort(all(pos));

        vector<mint> dp(k + 2);  // dp[i] := i番目のワープホールの入り口に到達する経路の数
        dp[0] = 1;
        reps(i, k + 1) {
            auto [ai, bi, ci, di] = pos[i];
            dp[i] = C((ai - 1) + (bi - 1), ai - 1);
            reps(j, i - 1) {
                auto [aj, bj, cj, dj] = pos[j];
                dp[i] -= dp[j] * (C((ai - aj) + (bi - bj), ai - aj) - C((ai - cj) + (bi - dj), ai - cj));
            }
        }

        cout << dp.back() << endl;
    }
}
```