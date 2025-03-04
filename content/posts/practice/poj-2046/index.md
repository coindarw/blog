+++
title = "POJ 2046 Gap(BOJ 3937)"
date = 2025-03-04T13:53:31+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=2046
https://www.acmicpc.net/problem/3937

https://vjudge.net/problem/POJ-2046
https://vjudge.net/problem/Baekjoon-3937
<!--more-->
## 問題概要
- 4x7=28枚のカードを使ってゲームを行う
- カードには2桁の数字が書いてあり、1桁目（1~4)はスート、2桁目(1~7)は価値を表す。

1. シャッフルして4行7列に並べる。左に1列分空きスペースを用意しておく
2. 2桁目が1のカード(11,21,31,41)を取って順に空きスペースに並べる。4つ穴ができる
3. 左にカードが存在する穴を1つ選ぶ。左のカードの番号を$X$としたとき、$X+1$のカードを取ってきて穴の部分に置く。27など$X+1$が書かれたカードが存在しない場合は何もしない
4. 手順3を繰り返して盤面を以下のようにする（XXは穴を表す）
```
12 13 14 15 16 17 21 XX
22 23 24 25 26 27 31 XX
32 33 34 35 36 37 41 XX
42 43 44 45 46 47 11 XX
```

- 最終的な盤面に持っていけるか判定、できる場合は最小手数（手順3を行う回数）を出力
## 解法メモ
- BFSをすると解ける

そこに至るまでの流れを書くと以下のような感じ
- 最初に愚直全探索を書く
- IDDFS+簡単な枝刈りにする
- 非常に遅いのでメモ化すると高速化、同じ状態をたくさん通っていることが分かる
- やっていることがただのBFSなのでDFSから書き換える
- 2重vectorのmapは重すぎて辛かったので3つのuint64変数に収める
	- 値の個数は穴も含めると29種類、5bitに収まる
	- 盤面は32マスなので、トータル5×32=160bit、64bit整数3つあれば収まる
## 実装例
```cpp
#include <iostream>
#include <map>
#include <queue>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
using namespace std;
#define all(v) v.begin(), v.end()

typedef unsigned long long ull;
typedef pair<int, int> P;

const int f[28 + 1] = {0, 11, 12, 13, 14, 15, 16, 17, 21, 22, 23, 24, 25, 26, 27, 31, 32, 33, 34, 35, 36, 37, 41, 42, 43, 44, 45, 46, 47};
ull rev_f[100];

struct State {
    ull data[3];
    int get(int i, int j) const {
        int idx = i * 8 + j;
        if (idx < 12) {
            int x = (data[0] >> (idx * 5)) & 31;
            return f[x];
        } else if (idx < 24) {
            idx -= 12;
            int x = (data[1] >> (idx * 5)) & 31;
            return f[x];
        }
        idx -= 24;
        int x = (data[2] >> (idx * 5)) & 31;
        return f[x];
    }
    void set(int i, int j, int x) {
        int idx = i * 8 + j;
        if (idx < 12) {
            data[0] &= ~((ull)31 << (idx * 5));
            data[0] |= rev_f[x] << (idx * 5);
        } else if (idx < 24) {
            idx -= 12;
            data[1] &= ~((ull)31 << (idx * 5));
            data[1] |= rev_f[x] << (idx * 5);
        } else {
            idx -= 24;
            data[2] &= ~((ull)31 << (idx * 5));
            data[2] |= rev_f[x] << (idx * 5);
        }
    }
    State() { data[0] = data[1] = data[2] = 0; }
    State(const vector<vector<int> >& a) {
        data[0] = data[1] = data[2] = 0;
        rep(i, 4) rep(j, 8) {
            int x = a[i][j];
            if (x < 0) continue;
            int idx = i * 8 + j;
            if (idx < 12) {
                data[0] |= rev_f[x] << (idx * 5);
            } else if (idx < 24) {
                idx -= 12;
                data[1] |= rev_f[x] << (idx * 5);
            } else {
                idx -= 24;
                data[2] |= rev_f[x] << (idx * 5);
            }
        }
    }
    bool operator<(const State& s) const {
        if (data[0] != s.data[0]) return data[0] < s.data[0];
        if (data[1] != s.data[1]) return data[1] < s.data[1];
        return data[2] < s.data[2];
    }
    bool operator==(const State& s) const { return data[0] == s.data[0] && data[1] == s.data[1] && data[2] == s.data[2]; }
};

void print(const State& a) {
    rep(i, 4) {
        rep(j, 8) { cerr << a.get(i, j) / 10 << a.get(i, j) % 10 << " "; }
        cerr << endl;
    }
    cerr << endl;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    rep(i, 28 + 1) rev_f[f[i]] = i;

    int t;
    cin >> t;
    while (t--) {
        State start, goal;
        rep(i, 4) reps(j, 7) {
            int x;
            cin >> x;
            start.set(i, j, x);
        }

        rep(i, 4) rep(j, 7) { goal.set(i, j, (i + 1) * 10 + (j + 1)); }

        rep(i, 4) { start.set(i, 0, (i + 1) * 10 + 1); }
        rep(i, 4) reps(j, 7) {
            if (start.get(i, j) % 10 == 1) {
                start.set(i, j, 0);
            }
        }

        vector<P> num_positions(100);
        queue<State> que;
        map<State, short> dist;
        dist[start] = 0;
        que.push(start);
        bool found = false;
        while (!que.empty()) {
            State curState = que.front();
            que.pop();

            if (curState == goal) {
                cout << dist[curState] << "\n";
                found = true;
                break;
            }
            rep(i, 4) rep(j, 8) { num_positions[curState.get(i, j)] = P(i, j); }
            rep(i, 4) reps(j, 7) {
                if (curState.get(i, j) != 0) continue;
                int left = curState.get(i, j - 1);
                if (left == 0) continue;
                if (left % 10 == 7) continue;
                State nxtState = curState;
                int ni = num_positions[left + 1].first;
                int nj = num_positions[left + 1].second;
                nxtState.set(i, j, left + 1);
                nxtState.set(ni, nj, 0);
                if (dist.find(nxtState) == dist.end()) {
                    dist[nxtState] = dist[curState] + 1;
                    que.push(nxtState);
                }
            }
        }
        if (!found) cout << -1 << "\n";
    }
}
```
