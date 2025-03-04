+++
title = "POJ 3526 The Teacher's Side of Math(AOJ 1284, BOJ 3904, Gym 101415J, UVa 1397)"
date = 2025-03-04T13:54:08+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3526
https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1284
https://codeforces.com/gym/101415/attachments
https://www.acmicpc.net/problem/3904
https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=4143

https://vjudge.net/problem/POJ-3526
https://vjudge.net/problem/Aizu-1284
https://vjudge.net/problem/Gym-101415J
https://vjudge.net/problem/Baekjoon-3904
https://vjudge.net/problem/UVA-1397
<!--more-->
## 問題概要
- 整数$a,m,b,n$が与えられます。以下の条件を満たす多項式 $F(x) = a_d x^d + a_{d-1} x^{d-1} + \cdots + a_1 x + a_0$ を求めてください
	-  係数$a_0,\cdots,a_q$は整数で、$a_q>0$
	- $f(\sqrt[m] a+\sqrt[n] b)=0$
	- 上2つの条件を満たす多項式の中で次数が最小
	- $\gcd(a_0, \cdots, a_q)=1$
### 制約
- マルチテストケース
- $a,b$が異なる素数
- $1<m,n$
- $\sqrt[m]{a}+\sqrt[n]{b}\leq 4$
- $mn \leq 20$
- 答えの係数 $a_0,\cdots,a_d$ は符号付き32bit整数の範囲に収まる

### ヒント
- 問題文に$a,b$が異なる素数かつ$1<m,n$なら答えの次数は$mn$であり、最上位項$a_d=1$であることが書いてある

## 解法メモ

- 入力にも出力にも$a$が登場してややこしいので、$F(x) = c_d x^d + c_{d-1} x^{d-1} + \cdots + c_1 x + c_0$とする

- $\sqrt[m] a+\sqrt[n] b$を代入すると以下のようになる
	- $c_d(\sqrt[m]{a}+\sqrt[n]{b})^d + c_{d-1}(\sqrt[m]{a}+\sqrt[n]{b})^{d-1} + \cdots + c_1(\sqrt[m]{a}+\sqrt[n]{b}) + c_0=0$
- これを全て展開し、$(\sqrt[m]{a})^i(\sqrt[n]{b})^j$の係数が0になるように連立方程式を立ててやればよい。
	- $c_d=1$であることが問題文からわかるので、$c_{d-1}, \cdots, c_0$の$d=mn$個の変数を用意してやり、各$0\leq i<m, 0\leq j<n$について$(\sqrt[m]{a})^i(\sqrt[n]{b})^j$の係数が0になるよう式を立てると、ちょうど$mn$次正方行列ができる。
- これをGauss-Jordan法等を用いて解いてやればよい。
## 実装例
最初整数だけで実装しようと思ったが、面倒だったので実数で計算した。
初回提出はlong doubleで実装したが、doubleでも普通に通る。

usingなど使っているのでPOJではCEになるので注意
```cpp
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

template <typename T>
struct Matrix : vector<T> {
    int n, m;
    Matrix(pair<int, int> p) : n(p.first), m(p.second) { this->resize(n * m); }
    Matrix(int n) : n(n), m(n) { this->resize(n * n); }
    Matrix() : n(0), m(0) {}
    T &operator()(int i, int j) { return (*this)[i * m + j]; }
};
template <typename T>
struct vec : vector<T> {
    int n;
    vec(int n) : n(n) { this->resize(n); }
    vec() : n(0) {}
};

template <typename T>
Matrix<T> new_matrix(int n) {
    Matrix<T> res({n, n});
    return res;
}
template <typename T>
Matrix<T> operator*(const Matrix<T> &m1, const Matrix<T> &m2) {
    assert(m1.m == m2.n);
    Matrix<T> res({m1.n, m2.m});
    rep(i, m1.n) rep(j, m2.m) rep(k, m1.m) { res[i * m2.m + j] += m1[i * m1.m + k] * m2[k * m2.m + j]; }
    return res;
}
template <typename T>
vec<T> operator*(const Matrix<T> &m, const vec<T> &v) {
    assert(m.m == v.n);
    vec<T> res(m.n);
    rep(i, m.n) {
        T sum = 0;
        rep(j, m.m) { sum += m[i * m.m + j] * v[j]; }
        res.push_back(sum);
    }
    return res;
}
template <typename T>
Matrix<T> E(size_t n) {
    Matrix<T> res({n, n});
    rep(i, n) res[i * n + i] = 1;
    return res;
}
template <typename T>
Matrix<T> pow(const Matrix<T> &a, long long n) {
    Matrix<T> b = a;
    Matrix<T> res = E(a.size());
    while (n > 0) {
        if (n & 1) res = res * b;
        b = b * b;
        n >>= 1ll;
    }
    return res;
}

template <typename T>
vec<T> gauss_jordan(const Matrix<T> &A, const vec<T> &b) {  // 連立一次方程式Ax=bを解く
    assert(A.n == A.m);
    assert(A.n == b.n);
    int n = A.n;
    const T eps = 1e-8;
    Matrix<T> B({n, n + 1});
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++) B[i * (n + 1) + j] = A[i * n + j];
    for (int i = 0; i < n; i++) B[i * (n + 1) + n] = b[i];

    for (int i = 0; i < n; i++) {
        int pivot = i;
        for (int j = i; j < n; j++) {
            if (abs(B[j * (n + 1) + i]) > abs(B[pivot * (n + 1) + i])) pivot = j;
        }
        for (int j = i; j < n + 1; ++j) swap(B[i * (n + 1) + j], B[pivot * (n + 1) + j]);
        if (abs(B[i * (n + 1) + i]) < eps) return vec<T>();
        for (int j = i + 1; j <= n; j++) B[i * (n + 1) + j] /= B[i * (n + 1) + i];
        for (int j = 0; j < n; j++) {
            if (i != j) {
                for (int k = i + 1; k <= n; k++) B[j * (n + 1) + k] -= B[j * (n + 1) + i] * B[i * (n + 1) + k];
                B[j * (n + 1) + i] = 0;
            }
        }
        B[i * (n + 1) + i] = 1;
    }
    vec<T> x(n);
    for (int i = 0; i < n; i++) x[i] = B[i * (n + 1) + n];
    return x;
}

template <typename T>
T binom(int n, int m) {  // パスカルの三角形
    if (n < m || n < 0 || m < 0) return 0;
    if (m == 0 || m == n) return 1;
    static vector<vector<T>> memo;
    static vector<vector<bool>> memoized;
    while ((int)memo.size() < n + 1) memo.push_back({}), memoized.push_back({});
    while ((int)memo[n].size() < m + 1) memo[n].push_back(0), memoized[n].push_back(false);
    if (memoized[n][m]) return memo[n][m];
    memoized[n][m] = true;
    return memo[n][m] = binom<T>(n - 1, m - 1) + binom<T>(n - 1, m);
}

int main() {
    while (true) {
        int a, m, b, n;
        cin >> a >> m >> b >> n;
        if (a == 0 && m == 0 && b == 0 && n == 0) break;
        Matrix<double> M({n * m, n * m});

        auto idx = [&](int i, int j) -> int { return i * n + j; };  // \sqrt[m]{a}^i+\sqrt[n]{b}^j

        rep(i, n * m) {      // x^iの係数を連立方程式の変数とする
            rep(j, i + 1) {  // (\sqrt[m]{a}+\sqrt[n]{b})^iを展開した時のj番目の係数
                double c = binom<double>(i, j) * pow(a, (i - j) / m) * pow(b, j / n);
                int id = idx((i - j) % m, j % n);
                M(id, i) += c;
            }
        }
        vec<double> v(n * m);  // 最上位項は1固定なのでMx=vのvの部分
        rep(j, n * m + 1) {
            double c = binom<double>(n * m, j) * pow(a, (n * m - j) / m) * pow(b, j / n);
            int id = idx((n * m - j) % m, j % n);
            v[id] -= c;
        }

        vec<double> res = gauss_jordan(M, v);
        assert(!res.empty());
        res.push_back(1);
        reverse(all(res));
        rep(i, n * m + 1) {
            cout << ll(roundl(res[i]));
            if (i != n * m) cout << " ";
        }
        cout << endl;
    }
}
```