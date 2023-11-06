+++
title = 'コピペ用 ACLib仕様の基本的な遅延セグメント木'
date = 2023-11-05T01:34:17+09:00
tags = ['遅延セグメント木', '競技プログラミング']
+++

使い方などは [AC-Libraryのドキュメント](https://atcoder.github.io/ac-library/document_ja/lazysegtree.html) を参照

コンテスト中、AOJ Coursesにあるようなやつはコピペして使いたいのでまとめ

<!--more-->

infはint64で表せる十分大きい整数で、足してもオーバーフローしないくらいの大きさを想定。普段は$4\times10^{18}$を使っている

## Range Affine Range Sum

[https://atcoder.jp/contests/practice2/tasks/practice2_k](https://atcoder.jp/contests/practice2/tasks/practice2_k)

*a+bの更新、区間Sum。適宜llをmintとかに変更

1倍してb足すのをRAQ、0倍してa足すのをRUQとして使える

```cpp
struct S {
    ll a;
    int size;
};

struct F {
    ll a, b;
};

S op(S l, S r) { return {l.a + r.a, l.size + r.size}; }

S e() { return {0, 0}; }

S mapping(F l, S r) { return {r.a * l.a + r.size * l.b, r.size}; }

F composition(F l, F r) { return {r.a * l.a, r.b * l.a + l.b}; }

F id() { return F{1, 0}; }
```

## Range Update Query (RUQ)

[https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_D&lang=ja](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_D&lang=ja)

```cpp
using S = ll;
struct F {
    bool id;  // updateに単位元を作るためのフラグ
    ll x;
};

S op(S l, S r) { return min(l, r); }

S e() { return inf; }

S mapping(F f, S s) { return f.id ? s : f.x; }

F composition(F f, F g) { return {f.id && g.id, f.id ? g.x : f.x}; }

F id() { return {true, 0ll}; }
```

## RMQ and RUQ

[https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_F&lang=ja](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_F&lang=ja)

Min Query

```cpp
using S = ll;
struct F {
    bool id;
    ll x;
};

S op(S l, S r) { return min(l, r); }

S e() { return inf; }

S mapping(F f, S s) { return f.id ? s : f.x; }

F composition(F f, F g) { return {f.id && g.id, f.id ? g.x : f.x}; }

F id() { return {true, 0ll}; }
```

## RSQ and RAQ

[https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_G&lang=ja](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_G&lang=ja)

```cpp
struct S {
    ll sum, size;
};
using F = ll;

S op(S l, S r) { return S{l.sum + r.sum, l.size + r.size}; }

S e() { return {0, 0}; }

S mapping(F f, S s) { return S{s.sum + s.size * f, s.size}; }

F composition(F f, F g) { return f + g; }

F id() { return 0; }
```

## RMQ and RAQ

[https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_H&lang=ja](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_H&lang=ja)

Min Query

```cpp
using S = ll;
using F = ll;

S op(S l, S r) { return min(l, r); }

S e() { return inf; }

S mapping(F f, S s) { return s == inf ? inf : s + f; }

F composition(F f, F g) { return f + g; }

F id() { return 0; }
```

## RSQ and RUQ

[https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_I&lang=ja](https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_2_I&lang=ja)

```cpp
struct S {
    ll sum, size;
};
struct F {
    bool id;
    ll x;
};

S op(S l, S r) { return S{l.sum + r.sum, l.size + r.size}; }

S e() { return S{0, 0}; }

S mapping(F f, S s) { return f.id ? s : S{f.x * s.size, s.size}; }

F composition(F f, F g) { return f.id ? g : f; }

F id() { return {true, 0ll}; }
```