+++
title = "POJ 2823 Sliding Window"
date = 2025-03-04T13:53:52+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2823

https://vjudge.net/problem/POJ-2823
<!--more-->
## 問題概要
- 長さ$N$の配列が与えられる。長さ$K$の連続部分列$N-K+1$個それぞれについて、最大値と最小値を出力
### 制約
- $N\leq 10^6$
## 解法メモ
- スライド最小値・最大値の練習問題
- SWAGでもOK
### 実装例
スライド最小値解。
Language G++だとcin/coutで通ってscanf/printfで通らない、C++だと逆にcin/coutだと通らなくてscanf/printfだと通った。
```cpp
#include <cstdio>
#include <deque>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, k;
    cin >> n >> k;

    vector<int> a(n);
    rep(i, n) cin >> a[i];

    deque<int> dq;
    rep(i, n) {
        while (!dq.empty() && a[dq.back()] >= a[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k)
            if (dq.front() == i - k) dq.pop_front();
        if (i >= k - 1) cout << a[dq.front()] << " ";
    }
    cout << "\n";
    dq.clear();
    rep(i, n) {
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k)
            if (dq.front() == i - k) dq.pop_front();
        if (i >= k - 1) cout << a[dq.front()] << " ";
    }
    cout << "\n";
}
```

SWAG解。
スライド最小値よりも数%~2割程度速かった。少し意外


なお、stackの内部実装としてvectorを指定している。デフォルトではdequeが使われるが、それに比べて2割くらい速くなる。
競プロでは再配置の際のコピーコストが問題になるようなオブジェクトを載せることはほとんどないため、基本的には何も考えずvector指定していいかもしれない。
```cpp
#include <iostream>
#include <stack>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

template <typename T, T (*op)(T, T)>
struct SWAG {
    stack<T, vector<T> > st1, st2, raw;
    void push(T x) {
        if (st1.empty()) st1.push(x);
        else st1.push(op(st1.top(), x));
        raw.push(x);
    }

    void move() {
        if (st2.empty()) {
            while (!st1.empty()) {
                st1.pop();
                if (st2.empty()) st2.push(raw.top());
                else st2.push(op(raw.top(), st2.top()));
                raw.pop();
            }
        }
    }

    void pop() {
        move();
        st2.pop();
    }

    T prod() {
        if (st2.empty()) return st1.top();
        if (st1.empty()) return st2.top();
        return op(st2.top(), st1.top());
    }

    int size() { return st1.size() + st2.size(); }
};

int op_min(int a, int b) { return min(a, b); }
int op_max(int a, int b) { return max(a, b); }

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, k;
    cin >> n >> k;

    vector<int> a(n);
    rep(i, n) cin >> a[i];

    SWAG<int, op_min> swag_min;
    SWAG<int, op_max> swag_max;
    rep(i, n) {
        swag_min.push(a[i]);
        if (i >= k) swag_min.pop();
        if (i >= k - 1) cout << swag_min.prod() << " ";
    }
    cout << "\n";
    rep(i, n) {
        swag_max.push(a[i]);
        if (i >= k) swag_max.pop();
        if (i >= k - 1) cout << swag_max.prod() << " ";
    }
    cout << "\n";
}
```