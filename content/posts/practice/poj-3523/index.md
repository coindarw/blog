+++
title = "POJ 3523 The Morning after Halloween(AOJ 1281,BOJ 3901,Gym 101415G)"
date = 2025-03-04T13:54:07+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3523
https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1281
https://www.acmicpc.net/problem/3901
https://codeforces.com/gym/101415/attachments

https://vjudge.net/problem/POJ-3523
https://vjudge.net/problem/Aizu-1281
https://vjudge.net/problem/Baekjoon-3901
https://vjudge.net/problem/Gym-101415G
<!--more-->

※AOJでAC取ったがPOJではTLEする
## 問題概要
- 下のような$h\times w$のグリッドが与えられる
- `a,b,c`はゴーストの位置、`A,B,C`は各ゴーストの最終的な目的地を表す
- 各ゴーストが目的地に到着するのに何ステップかかるか
- 各ステップで各ゴーストは4近傍のいずれかに移動するか同じマスにとどまることができるが、以下のような移動はできない
	- `#`の場所は通れない
	- 移動後に同じ位置に重なってはならない
	- 2体の位置が入れ替わる（すれ違う）ような移動はできない
```
#######
#A#B#C#
#     #
#b#a#c#
#######
```

### 制約
- $4\leq w\leq 16$
- $4\leq h\leq 16$
- ゴーストの数$1\leq n\leq 3$
- 目的地にたどり着けない入力は与えられない

## 解法メモ
A\*を実装した。
ヒューリスティック関数としては、`a,b,c`からゴールへの距離の平均値を設定した

## 実装例
最初距離をmapで保存していたが、TLEするのでvectorにした
MLスレスレになるのでshort型にしている


```cpp
#include <cstdio>
#include <iostream>
#include <map>
#include <queue>
#include <set>
#include <sstream>
#include <string>
#include <vector>
typedef long long ll;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define inf int(2e9)
using namespace std;

typedef pair<int, int> P;
const int dx[] = {1, 0, -1, 0, 0};
const int dy[] = {0, 1, 0, -1, 0};

int n;
vector<string> v;
vector<vector<vector<int> > > dist(3);

int h_star(int idx) {
    int ai = idx / 256 / 256, bi = idx / 256 % 256, ci = idx % 256;
    int ay = ai / 16, ax = ai % 16;
    int by = bi / 16, bx = bi % 16;
    int cy = ci / 16, cx = ci % 16;
    int res = 0;
    if (n == 2) {
        res = dist[0][ay][ax] + dist[1][by][bx];
    } else if (n == 3) {
        res = dist[0][ay][ax] + dist[1][by][bx] + dist[2][cy][cx];
    }
    return res / n;
}

int encode(int ay, int ax, int by, int bx, int cy, int cx) {
    int ai = ay * 16 + ax;
    int bi = by * 16 + bx;
    int ci = cy * 16 + cx;
    return ai * 256 * 256 + bi * 256 + ci;
}

// デバッグ用
string toString(int idx) {
    int ai = idx / 256 / 256, bi = idx / 256 % 256, ci = idx % 256;
    int ay = ai / 16, ax = ai % 16;
    int by = bi / 16, bx = bi % 16;
    int cy = ci / 16, cx = ci % 16;
    stringstream ss;
    vector<string> vs = v;
    if (n >= 1) vs[ay][ax] = 'a', ss << "a: " << ay << " " << ax << endl;
    if (n >= 2) vs[by][bx] = 'b', ss << "b: " << by << " " << bx << endl;
    if (n >= 3) vs[cy][cx] = 'c', ss << "c: " << cy << " " << cx << endl;
    rep(i, vs.size()) { ss << vs[i] << endl; }
    return ss.str();
}

vector<int> next_steps(int idx) {
    int ai = idx / 256 / 256, bi = idx / 256 % 256, ci = idx % 256;
    int ay = ai / 16, ax = ai % 16;
    int by = bi / 16, bx = bi % 16;
    int cy = ci / 16, cx = ci % 16;
    vector<int> res;
    if (n == 2) {
        rep(i, 5) {
            int nay = ay + dy[i], nax = ax + dx[i];
            if (nay < 0 || nay >= v.size() || nax < 0 || nax >= v[0].size()) continue;
            if (v[nay][nax] == '#') continue;
            rep(j, 5) {
                int nby = by + dy[j], nbx = bx + dx[j];
                if (nby < 0 || nby >= v.size() || nbx < 0 || nbx >= v[0].size()) continue;
                if (v[nby][nbx] == '#') continue;
                if (nay == by && nax == bx && nby == ay && nbx == ax) continue;
                if (nay == nby && nax == nbx) continue;
                res.push_back(encode(nay, nax, nby, nbx, cy, cx));
            }
        }
    } else if (n == 3) {
        rep(i, 5) {
            int nay = ay + dy[i], nax = ax + dx[i];
            if (nay < 0 || nay >= v.size() || nax < 0 || nax >= v[0].size()) continue;
            if (v[nay][nax] == '#') continue;
            rep(j, 5) {
                int nby = by + dy[j], nbx = bx + dx[j];
                if (nby < 0 || nby >= v.size() || nbx < 0 || nbx >= v[0].size()) continue;
                if (v[nby][nbx] == '#') continue;
                if (nay == by && nax == bx && nby == ay && nbx == ax) continue;
                if (nay == nby && nax == nbx) continue;
                rep(k, 5) {
                    int ncy = cy + dy[k], ncx = cx + dx[k];
                    if (ncy < 0 || ncy >= v.size() || ncx < 0 || ncx >= v[0].size()) continue;
                    if (v[ncy][ncx] == '#') continue;
                    if (nay == cy && nax == cx && ncy == ay && ncx == ax) continue;
                    if (nby == cy && nbx == cx && ncy == by && ncx == bx) continue;
                    if (nay == ncy && nax == ncx) continue;
                    if (nby == ncy && nbx == ncx) continue;
                    res.push_back(encode(nay, nax, nby, nbx, ncy, ncx));
                }
            }
        }
    }
    return res;
}

template <typename State, typename Cost, vector<State> (*next_steps)(State), Cost (*h_star)(State)>
Cost a_star(State start, State goal) {
    typedef pair<Cost, State> CS;
    priority_queue<CS, vector<CS>, greater<CS> > pq;
    vector<short> dist(256 * 256 * 256, 10000);
    set<State> seen;
    dist[start] = 0;
    pq.push(make_pair(h_star(start), start));
    while (!pq.empty()) {
        CS u = pq.top();
        Cost curDist = u.first;
        State curPos = u.second;
        pq.pop();
        if (curPos == goal) {
            return curDist;
        }
        if (seen.find(curPos) != seen.end()) continue;
        seen.insert(curPos);
        vector<State> nxt = next_steps(curPos);
        Cost gn = curDist - h_star(curPos);
        for (size_t i = 0; i < nxt.size(); ++i) {
            State nxtState = nxt[i];
            Cost cost = 1;
            Cost nxtDist = gn + h_star(nxtState) + cost;
            if (dist[nxtState] > nxtDist) {
                dist[nxtState] = nxtDist;
                pq.push(make_pair(nxtDist, nxtState));
            }
        }
    }
    return dist[goal];
};

vector<vector<int> > grid_bfs(const vector<string>& v, int sy, int sx) {
    const int h = v.size(), w = v.at(0).size();
    queue<P> que;
    vector<vector<int> > dist(h, vector<int>(w, inf));
    dist[sy][sx] = 0;
    que.push(make_pair(sy, sx));
    while (!que.empty()) {
        P p = que.front();
        int y = p.first, x = p.second;
        que.pop();
        rep(i, 4) {
            int ny = y + dy[i], nx = x + dx[i];
            if (ny < 0 || ny >= h || nx < 0 || nx >= w) continue;
            if (v[ny][nx] == '#') continue;
            if (dist[ny][nx] > dist[y][x] + 1) {
                dist[ny][nx] = dist[y][x] + 1;
                que.push(make_pair(ny, nx));
            }
        }
    }
    return dist;
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    while (true) {
        int w, h;
        cin >> w >> h >> n;
        if (w == 0 && h == 0 && n == 0) break;
        v.resize(h);
        cin.ignore();
        rep(i, h) { getline(cin, v[i]); }

        vector<P> sPos(3), gPos(3);
        rep(i, h) rep(j, w) {
            if (v[i][j] == 'a') sPos[0] = make_pair(i, j), v[i][j] = ' ';
            else if (v[i][j] == 'b') sPos[1] = make_pair(i, j), v[i][j] = ' ';
            else if (v[i][j] == 'c') sPos[2] = make_pair(i, j), v[i][j] = ' ';
            else if (v[i][j] == 'A') gPos[0] = make_pair(i, j);
            else if (v[i][j] == 'B') gPos[1] = make_pair(i, j);
            else if (v[i][j] == 'C') gPos[2] = make_pair(i, j);
        }

        dist.resize(n);
        rep(i, n) dist[i] = grid_bfs(v, gPos[i].first, gPos[i].second);

        if (n == 1) {  // n=1の場合は普通のBFS
            cout << dist[0][sPos[0].first][sPos[0].second] << endl;
        } else {  // n=2,3の場合はA*を使う
            int start = encode(sPos[0].first, sPos[0].second, sPos[1].first, sPos[1].second, sPos[2].first, sPos[2].second);
            int goal = encode(gPos[0].first, gPos[0].second, gPos[1].first, gPos[1].second, gPos[2].first, gPos[2].second);
            cout << a_star<int, int, next_steps, h_star>(start, goal) << endl;
        }
    }
}
```