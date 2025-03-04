+++
title = "POJ 2345 Central heating(EOlymp 219)"
date = 2025-03-04T13:53:41+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2345
https://basecamp.eolymp.com/en/problems/219

https://vjudge.net/problem/POJ-2345
https://vjudge.net/problem/EOlymp-219
<!--more-->
## 問題概要
- 大学には$N$人の技術者がおり、$N$個のバルブがある。
- 各技術者は1つ以上のバルブを担当しており、指示を受けるとバルブを操作する。つまり、開いているバルブは閉じ、閉じているバルブは開ける。
	- あるバルブを担当する技術者が複数いる場合もある
- ある技術者を他の技術者の組み合わせに置き換えることはできない
- 全てのバルブを開けるために必要な技術者のリストの内、最短のものを出力せよ

### 制約
- $1\leq N\leq 250$

## 解法メモ
- 各技術者に指示を送るかどうかを変数とし、各バルブについて足し算の代わりにXORを使う式を立てると$N$次正方行列ができるので、XORの掃き出し法を使うと解ける
## 実装例
自作bitsetを使っているので少し長いが、std::bitsetを使えばもっと短くなる

```cpp
#include <algorithm>
#include <cassert>
#include <cstdio>
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

struct BitSet {
    typedef unsigned long long ui;
    vector<ui> bits;
    int sz;
    BitSet() {}
    BitSet(int sz) : sz(sz) { bits = vector<ui>((sz + 63) / 64); }
    BitSet(int sz, const ui& val) : sz(sz) { bits = vector<ui>((sz + 63) / 64, val); }
    bool operator[](int i) const { return bits[i / 64] >> (i % 64) & 1; }
    void set(int i, bool val = true) {
        if (val) bits[i / 64] |= 1ull << (i % 64);
        else bits[i / 64] &= ~(1ull << (i % 64));
    }

    BitSet operator~() const {
        BitSet res(sz);
        for (int i = 0; i < int(bits.size()); i++) res.bits[i] = ~bits[i];
        return res;
    }
    BitSet& operator&=(const BitSet& rhs) {
        for (int i = 0; i < int(bits.size()); i++) bits[i] &= rhs.bits[i];
        return *this;
    }
    BitSet& operator|=(const BitSet& rhs) {
        for (int i = 0; i < int(bits.size()); i++) bits[i] |= rhs.bits[i];
        return *this;
    }
    BitSet& operator^=(const BitSet& rhs) {
        for (int i = 0; i < int(bits.size()); i++) bits[i] ^= rhs.bits[i];
        return *this;
    }
    BitSet& operator+=(const BitSet& rhs) {
        ui carry = 0;
        for (int i = 0; i < int(bits.size()); i++) {
            ui tmp = bits[i];
            bits[i] += rhs.bits[i] + carry;
            carry = (bits[i] < tmp) || (carry && bits[i] == tmp);
        }
        return *this;
    }
    BitSet& operator<<=(size_t n) {
        assert(n < 64);
        if (bits.empty() || n == 0) return *this;
        for (int i = int(bits.size()) - 1; i >= 0; i--) {
            bits[i] <<= n;
            if (i) bits[i] |= bits[i - 1] >> (64 - n);
        }
        return *this;
    }
    BitSet operator&(const BitSet& rhs) const { return BitSet(*this) &= rhs; }
    BitSet operator|(const BitSet& rhs) const { return BitSet(*this) |= rhs; }
    BitSet operator^(const BitSet& rhs) const { return BitSet(*this) ^= rhs; }
    BitSet operator+(const BitSet& rhs) const { return BitSet(*this) += rhs; }
    BitSet operator<<(size_t n) const { return BitSet(*this) <<= n; }

    friend ostream& operator<<(ostream& os, const BitSet& bs) {
        for (int i = 0; i < int(bs.sz); i++) os << bs[i];
        return os;
    }
    int count() const {
        int res = 0;
        for (int i = 0; i < int(bits.size()); i++) res += __builtin_popcountll(bits[i]);
        return res;
    }
};

vector<BitSet> gauss_jordan_xor(const vector<BitSet>& M) {
    const int n = M.size();
    const int m = M.front().sz;
    vector<BitSet> cM = M;
    int p = 0;
    for (int i = 0; i < m; ++i) {
        for (int j = i; j < n; ++j) {
            if (cM[j][i]) {
                swap(cM[j], cM[p]);
                break;
            }
        }
        if (p < n && cM[p][i]) {
            for (int j = 0; j < n; ++j) {
                if (j != p && cM[j][i]) {
                    cM[j] ^= cM[p];
                }
            }
            p++;
        }
    }
    return cM;
};

int main() {
    int n;
    cin >> n;
    vector<BitSet> a(n, BitSet(n + 1));
    rep(i, n) {
        while (true) {
            int x;
            cin >> x;
            if (x == -1) break;
            a[x - 1].set(i, true);
        }
    }
    rep(i, n) a[i].set(n, true);
    vector<BitSet> b = gauss_jordan_xor(a);
    rep(i, n) {
        if (b[i][n]) cout << i + 1 << " ";
    }
    cout << endl;
    return 0;
}
```
