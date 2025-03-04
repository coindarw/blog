+++
title = "POJ 3532 Resistance"
date = 2025-03-04T13:54:10+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3532

https://vjudge.net/problem/POJ-3532
<!--more-->
## 問題概要
- ある電気回路を表す$N$頂点$M$辺の単純とは限らない無向グラフが与えられる。
- 各辺の情報は$X_i,Y_i,R_i$で表され、これは頂点$X_i, Y_i$の間に$R_i$オームの抵抗があることを表す。
- ノード1とノード$N$の間の抵抗はいくらになるか求めよ
### 制約
- $N,M\leq 100$
- 問題文を見ても抵抗値の制約がないが、おそらくそんなに大きくない正整数
## 解法メモ
- 高校物理を覚えていなくて難しかった

- 各ノードに入る電流と出る電流が等しくなるはずなのでそれらについて式を立てていけばよさそう
- 変数として考えられるのは各ノードの電位か各辺を通る電流あたりとあたりをつける

- 電位を変数としてみる
	- 各ノードの電位を$v_i$、辺$\{i,j\}$間の抵抗を$R_{i,j}$とする。
	- 頂点1に頂点2,3,4が隣接しているとすると以下のような式が立てられる．これは線形なので掃き出し法で解けそう
	$$\dfrac{v_2-v_1}{R_{1,2}}+\dfrac{v_3-v_1}{R_{1,3}}+\dfrac{v_4-v_1}{R_{1,4}}=0$$


## 実装例
行列まわりの構造体・関数が上手く書けなくて試行錯誤の跡が見える実装。
ガウス・ジョルダンの係数行列が正方行列でない場合でも引数としては受け付けるが、動かすと壊れるので注意
```cpp
#include <algorithm>
#include <cassert>
#include <cmath>
#include <cstdio>
#include <iomanip>
#include <iostream>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define inf int(2e9)
using namespace std;
#define all(v) v.begin(), v.end()

template <typename T>
struct Matrix : vector<T> {
    int n, m;
    Matrix(int n, int m) : n(n), m(m) { this->resize(n * m); }
    Matrix(int n) : n(n), m(n) { this->resize(n * n); }
    Matrix() : n(0), m(0) {}
    T &operator()(int i, int j) { return (*this)[i * m + j]; }
    const T &operator()(int i, int j) const { return (*this)[i * m + j]; }
};
template <typename T>
struct vec : vector<T> {
    int n;
    vec(int n) : n(n) { this->resize(n); }
    vec() : n(0) {}
};

template <typename T>
Matrix<T> new_matrix(int n) {
    Matrix<T> res(n);
    return res;
}
template <typename T>
Matrix<T> operator*(const Matrix<T> &m1, const Matrix<T> &m2) {
    assert(m1.m == m2.n);
    Matrix<T> res(m1.n, m2.m);
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
    Matrix<T> res(n);
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
    assert(A.n >= A.m);
    assert(A.n == b.n);
    int n = A.n, m = A.m;
    const T eps = 1e-8;
    Matrix<T> B(n, m + 1);
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++) B(i, j) = A(i, j);
    for (int i = 0; i < n; i++) B(i, m) = b[i];

    for (int i = 0; i < n; i++) {
        int pivot = i;
        for (int j = i; j < n; j++) {
            if (abs(B(j, i)) > abs(B(pivot, i))) pivot = j;
        }
        for (int j = i; j < m + 1; ++j) swap(B(i, j), B(pivot, j));
        if (abs(B(i, i)) < eps) return vec<T>();
        for (int j = i + 1; j < m + 1; j++) B(i, j) /= B(i, i);
        for (int j = 0; j < n; j++) {
            if (i != j) {
                for (int k = i + 1; k < m + 1; k++) B(j, k) -= B(j, i) * B(i, k);
                B(j, i) = 0;
            }
        }
        B(i, i) = 1;
    }
    vec<T> res(n);
    for (int i = 0; i < n; i++) res[i] = B(i, m);
    return res;
}

struct edge {
    int to;
    int resistance;
    edge(int to, int resistance) : to(to), resistance(resistance) {}
};
int main() {
    int n, m;
    cin >> n >> m;
    Matrix<double> A(n, n);
    vector<vector<edge> > G(n);
    rep(i, m) {
        int x, y, r;
        cin >> x >> y >> r;
        x--, y--;
        G[x].push_back(edge(y, r));
        G[y].push_back(edge(x, r));
    }

    rep(i, n) {
        if (i == 0 || i == n - 1) continue;
        for (int j = 0; j < G[i].size(); ++j) {
            edge e = G[i][j];
            A(i, i) -= 1.0 / e.resistance;
            A(i, e.to) += 1.0 / e.resistance;
        }
    }
    vec<double> b(n);
    A(0, 0) = 1;
    b[0] = 0;  // 最初の頂点は0V
    A(n - 1, n - 1) = 1;
    b[n - 1] = 1;  // 最後の頂点は1V

    vec<double> res = gauss_jordan(A, b);
    double current = 0;
    for (int j = 0; j < G[0].size(); ++j) {
        edge e = G[0][j];
        current += res[e.to] / e.resistance;
    }
    cout << fixed << setprecision(2);
    cout << 1. / current << endl;
}
```