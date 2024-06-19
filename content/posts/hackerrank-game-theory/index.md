+++
title = 'Hackerrank Game Theory'
date = 2024-06-19T17:38:15+09:00
tags = ['ゲーム問題', '競技プログラミング']
+++

5 Days of Game Theoryというものがかつてあったらしいのでやる
https://www.hackerrank.com/5-days-of-game-theory

vjudge
https://vjudge.net/problem#OJId=HackerRank&probNum=&title=&source=5-days-of-game-theory&category=all


<!--more-->

特に記述がない場合は問題文に以下が付いているものとする。
	2人のプレイヤーがゲームをする。プレイヤー1が先手で、交互に手番が回る．両者最適に行動した場合、どちらが勝つか判定せよ
	テストケース数をTとする
## Day 1: Game of Stones
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/a-game-of-stones
### 問題概要
T<=100

山に石がN(<=100)個あり、各手番で2,3,5個のいずれか取り除く。できない場合負ける。
### 解法メモ
DPやメモ化再帰で山の数がiであるとき勝ちか負けかを調べる
### 実装例
```cpp
// Day1 Game of Stones
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    vector<bool> dp(101);
    rep(i, 100) {
        if (i + 2 <= 100) dp[i + 2] = dp[i + 2] || !dp[i];
        if (i + 3 <= 100) dp[i + 3] = dp[i + 3] || !dp[i];
        if (i + 5 <= 100) dp[i + 5] = dp[i + 5] || !dp[i];
    }
    rep(_, t) {
        int n;
        cin >> n;
        cout << (dp[n] ? "First" : "Second") << "\n";
    }
}
```
## Day 1: Tower Breakers
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/tower-breakers
### 問題概要
T<=100

塔がN(<=10^6)個あり、それぞれの高さはM(<=10^6)である。

各手番で1つ塔を選び、その高さを現在の値の約数に変える。（約数として現在の値を選ぶことは不可）

全ての塔の高さが1の状態で手番が回ってくると負け。
### 解法メモ
Nimとみなせる（各塔が山、素因数の個数が石の個数）ので、Nが偶数 or M=1 なら後手の勝ち、それ以外なら先手の勝ち

### 実装例
```cpp
// Day1 Tower Breakers
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n, m;
        cin >> n >> m;
        cout << ((n % 2 == 0 || m == 1) ? 2 : 1) << "\n";
    }
}
```
## Day 1: A Chessboard Game
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/day-1-a-chessboard-game
### 問題概要
T<=15x15

15x15のチェス盤でゲームを行う。最初(xi, yi)にコインが置かれている

各手番でコインを(x-2,y+1),(x-2,y-1),(x+1,y-2),(x-1,y-2)に移動できる（ナイトの動ける場所の内半分。問題文の図参照）
### 解法メモ
メモ化再帰でコインが(i,j)にいるとき勝ちか負けかを調べる。トポロジカル順序が面倒なので再帰で実装するのが楽。
### 実装例
```cpp
// Day1 A Chessboard Game
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    vector<vector<int>> memo = vector<vector<int>>(15, vector<int>(15, -1));

    auto rec = [&](auto rec, int x, int y) {
        if (memo[x][y] != -1) return memo[x][y];
        int res = 0;
        if (x - 2 >= 0 && y + 1 < 15) res |= !rec(rec, x - 2, y + 1);
        if (x - 2 >= 0 && y - 1 >= 0) res |= !rec(rec, x - 2, y - 1);
        if (x + 1 < 15 && y - 2 >= 0) res |= !rec(rec, x + 1, y - 2);
        if (x - 1 >= 0 && y - 2 >= 0) res |= !rec(rec, x - 1, y - 2);
        return memo[x][y] = res;
    };

    int t;
    cin >> t;
    rep(_, t) {
        int x, y;
        cin >> x >> y;
        cout << (rec(rec, x - 1, y - 1) ? "First" : "Second") << "\n";
    }
}
```
## Day 2: Nim Game
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/day-2-nim-game
### 問題概要
T<=100

n個(<=100)の山があり、各山にはs_i個(<=100)個の石がある。各手番で1つの山から1つ以上の石を取り除く。全ての山が0個の状態で手番が回ってきたら負け
### 解法メモ
普通のNimなのでs_iのXORを取って0か判定すればよい
### 実装例
```cpp
// Day2 Nim Game
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        int all_xor = 0;
        rep(i, n) {
            int s;
            cin >> s;
            all_xor ^= s;
        }
        cout << (all_xor != 0 ? "First" : "Second") << "\n";
    }
}
```
## Day 2: Misère Nim
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/misere-nim
### 問題概要
T<=100

最後に石を取ると勝ちになるNim

n(<=100)個の山があり、各山にはs_i(<=10^9)個の石がある。各手番で1つの山から1つ以上の石を取り除く。最後の1つの石を取ると負け
### 解法メモ
競プロを始める前から何かを見て知っていた。

「全ての山の石の数が1個ならnが偶数→先手勝ち：nが奇数→後手勝ち、そうでなければ普通のNimと同じ」で勝ち負けを判定できる。

戦略：

基本的には普通のNimと同じように動き、自分の手番ですべての山の石の数が1以下になる時だけ、総XORが1になるように石を取ればよい。

（初期配置の時点ですべての山が1の場合を除いて、勝てる初期配置で最適に動いていれば、このような状況はゲーム中1回だけ自分の手番で現れる）

### 実装例
```cpp
// Day 2 Mise`re Nim
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        int all_xor = 0;
        bool all_one = true;
        rep(i, n) {
            int y;
            cin >> y;
            all_xor ^= y;
            all_one &= y == 1;
        }
        if (all_one) cout << (n % 2 == 0 ? "First" : "Second") << "\n";
        else cout << (all_xor != 0 ? "First" : "Second") << "\n";
    }
}
```
## Day 2: Nimble Game
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/nimble
### 問題概要
T<=10^4 

n(<=100)個のマス目があり、各マスにはc_i(<=10^9)個のコインがある。

各手番で1つマスを選び、そこからちょうど1つのコインを、それよりも番号が小さいマスに移動させる。操作ができない場合負け。
### 解法メモ
Nimとみなせる（各コインが山、コインのあるマス番号が石の数）。各ciは偶奇しか関係ないことに注意

### 実装例
```cpp
// Day2 Nimble Game
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        vector<int> c(n);
        rep(i, n) cin >> c[i];

        int all_xor = 0;
        rep(i, n) { all_xor ^= (c[i] % 2) * i; }

        cout << (all_xor != 0 ? "First" : "Second") << "\n";
    }
}
```
## Day 2: Poker Nim
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/day-2-poker-nim
### 問題概要
T<=100

n(<=100)個のチップの山があり、各山にはc_i(<=10^9)個のチップがある。

各手番で1つ山を選び、「1つ以上チップを取り除く」「1つ以上チップを追加する」どちらかの行動をする。ただし、追加は各プレイヤーごとにK(<=100)回しかできない。

操作ができない場合負け。
### 解法メモ
一瞬難しいと思ったが気づくとギャグ寄り。


チップ追加ルールを考えない、普通のNimの勝者をプレイヤーA、敗者をプレイヤーBとする。

もしお互いにチップを追加しなければ普通にNimをして終わる。したがって、最初にチップ追加するのはプレイヤーB。しかし、次の手番でプレイヤーAが追加分をそのまま取り除けば元の状態に戻ってしまう。

この行動にチップ追加回数は消費しないため、プレイヤーBの追加回数だけが減り、最終的に普通のNimになる。
### 実装例
```cpp
// Day2 Poker Nim
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int t;
    cin >> t;
    rep(_, t) {
        int n, k;
        cin >> n >> k;
        vector<int> a(n);
        rep(i, n) cin >> a[i];

        int all_xor = 0;
        rep(i, n) all_xor ^= a[i];

        cout << (all_xor != 0 ? "First" : "Second") << "\n";
    }
}
```


## Day 2: Tower Breakers, Revisited!
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/tower-breakers-2
### 問題概要
※Day1 Tower Breakersの塔の高さが一定でないver.


T<=100

塔がN(<=10^6)個あり、それぞれの高さはh_i(<=10^6)

各手番で1つ塔を選び、その高さを現在の値の約数に変える。（約数として現在の値を選ぶことは不可）

全ての塔の高さが1の状態で手番が回ってくると負け。
### 解法メモ
Nimとみなせる（各塔が山、素因数の個数が石の個数）ので、素因数分解してXORを求めればよい。
### 実装例
```cpp
// Day2 Tower Breakers, Revisited!
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

vector<int> prime_factorize(int n) {
    if (n <= 1) return {};
    vector<int> ans;
    for (int i = 2; i * i <= n; i++) {
        while (n % i == 0) {
            ans.push_back(i);
            n /= i;
        }
    }
    if (n != 1) {
        ans.push_back(n);
    }
    return ans;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        vector<int> a(n);
        rep(i, n) cin >> a[i];

        int all_xor = 0;
        rep(i, n) all_xor ^= prime_factorize(a[i]).size();

        cout << (all_xor != 0 ? 1 : 2) << "\n";
    }
}
```

## Day 3: Tower Breakers, Again!
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/tower-breakers-3
### 問題概要
T<=200

塔がN(<=100)個あり、それぞれの高さはh_i(<=10^5)

各手番で1つ塔を選ぶ。選んだ等の高さをXとして、Xの約数Yを1つ選ぶ。塔をY
個の高さX/Yの塔に分割する。


全ての塔の高さが1の状態で手番が回ってくると負け。

### 解法メモ
状態が分裂するタイプのゲームなので、再帰やDPで各高さのGrundy数を求めてXORを取ればよい。
### 実装例
```cpp
// Day3 Tower Breakers, Again!
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

vector<int> divisor(int n) {
    vector<int> res;
    for (int i = 1; i * i <= n; i++) {
        if (n % i == 0) {
            res.push_back(i);
            if (n / i != i) res.push_back(n / i);
        }
    }
    sort(res.begin(), res.end());
    return res;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    map<int, int> memo;
    auto grundy = [&](auto grundy, int h) {
        if (memo.count(h)) return memo[h];
        auto divs = divisor(h);
        vector<int> nxt;
        for (auto d : divs) {
            if (d == h) continue;
            int num = h / d;
            if (num % 2 == 0) {
                nxt.push_back(0);
            } else {
                nxt.push_back(grundy(grundy, d));
            }
        }
        sort(all(nxt));
        nxt.erase(unique(all(nxt)), nxt.end());
        rep(i, nxt.size()) {
            if (i != nxt[i]) return memo[h] = i;
        }
        return memo[h] = nxt.size();
    };

    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        vector<int> h(n);
        rep(i, n) cin >> h[i];

        int all_xor = 0;
        rep(i, n) all_xor ^= grundy(grundy, h[i]);

        cout << (all_xor != 0 ? 1 : 2) << "\n";
    }
}
```

## Day 3: Chessboard Game, Again! 
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/a-chessboard-game
### 問題概要
※Day1 A Chessboard Gameが、テストケース数225→1000、コインの数1→K(1000) に制約強化されたver.


T<=1000

15x15のチェス盤でゲームを行う。K(<=1000)個のコインが置かれており、それぞれ(x_i, y_i)にある

各手番でコインを(x-2,y+1),(x-2,y-1),(x+1,y-2),(x-1,y-2)に移動できる（ナイトの動ける場所の内半分。問題文の図参照）

コイン同士は重なっても構わない。
### 解法メモ
メモ化再帰でコインが(i,j)にいるときのGrundy数を調べてXORを取ればよい。
### 実装例
```cpp
// Day3 Chessboard Game, Again!
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    vector<vector<int>> memo = vector<vector<int>>(15, vector<int>(15, -1));

    auto rec = [&](auto rec, int x, int y) {
        if (memo[x][y] != -1) return memo[x][y];
        vector<int> nxt;
        if (x - 2 >= 0 && y + 1 < 15) nxt.push_back(rec(rec, x - 2, y + 1));
        if (x - 2 >= 0 && y - 1 >= 0) nxt.push_back(rec(rec, x - 2, y - 1));
        if (x + 1 < 15 && y - 2 >= 0) nxt.push_back(rec(rec, x + 1, y - 2));
        if (x - 1 >= 0 && y - 2 >= 0) nxt.push_back(rec(rec, x - 1, y - 2));
        sort(all(nxt));
        nxt.erase(unique(all(nxt)), nxt.end());
        rep(i, nxt.size()) {
            if (i != nxt[i]) return memo[x][y] = i;
        }
        return memo[x][y] = nxt.size();
    };

    int t;
    cin >> t;
    rep(_, t) {
        int k;
        cin >> k;
        int all_xor = 0;
        rep(i, k) {
            int x, y;
            cin >> x >> y;
            all_xor ^= rec(rec, x - 1, y - 1);
        }
        cout << (all_xor != 0 ? "First" : "Second") << "\n";
    }
}
```
## Day 3: Digits Square Board
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/digits-square-board
### 問題概要
T<=10

NxNのグリッドがあり、各マスには整数1...9までの整数A_{i,j}が書かれている。（N<=30）

各手番で、プレイヤーは非素数が書かれているグリッドを1つ選び、縦か横に2分割する。（板チョコのような感じ）

操作ができない場合、つまり全てのグリッドが「素数しか書かれていない」or「非素数が書かれていて1x1」のどちらかになった状態で手番が回ってきた場合負け。


非素数には1も含むことに注意
### 解法メモ
Grundy数を定義通りに求めていけばよい。グリッドの左端・右端・上端・下端・どこで切るか を探索するのでO(N^5)

非素数が含まれるかどうかは2次元imos法などを使えばよい
### 実装例
```cpp
// Day3 Digits Square Board
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

bool isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            return false;
        }
    }
    return true;
}

int memo[31][31][31][31];
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        vector<vector<int>> a(n + 1, vector<int>(n + 1));
        reps(i, n) reps(j, n) cin >> a[i][j];

        vector<vector<int>> ca(n + 1, vector<int>(n + 1));
        reps(i, n) reps(j, n) ca[i][j] = !isPrime(a[i][j]);

        reps(i, n) reps(j, n) ca[i][j] += ca[i - 1][j];
        reps(i, n) reps(j, n) ca[i][j] += ca[i][j - 1];

        memset(memo, -1, sizeof(memo));
        auto grundy = [&](auto grundy, int y1, int x1, int y2, int x2) -> int {
            if (memo[y1][x1][y2][x2] != -1) return memo[y1][x1][y2][x2];
            int sum = ca[y2][x2] - ca[y1 - 1][x2] - ca[y2][x1 - 1] + ca[y1 - 1][x1 - 1];
            if (sum == 0) return memo[y1][x1][y2][x2] = 0;

            vector<int> nxt;
            for (int y = y1; y < y2; ++y) {
                nxt.push_back(grundy(grundy, y1, x1, y, x2) ^ grundy(grundy, y + 1, x1, y2, x2));
            }
            for (int x = x1; x < x2; ++x) {
                nxt.push_back(grundy(grundy, y1, x1, y2, x) ^ grundy(grundy, y1, x + 1, y2, x2));
            }

            sort(all(nxt));
            nxt.erase(unique(all(nxt)), nxt.end());
            rep(i, nxt.size()) {
                if (i != nxt[i]) return memo[y1][x1][y2][x2] = i;
            }
            return memo[y1][x1][y2][x2] = nxt.size();
        };

        cout << (grundy(grundy, 1, 1, n, n) != 0 ? "First" : "Second") << "\n";
    }
}
```
## Day 4: Fun Game
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/fun-game
### 問題概要
T<=10

長さN(<=1000)の正整数列A,Bがある。各要素A_i,B_i<=10^5。

各手番でプレイヤーは今までに選ばれていないiを選択、先手ならA_i点、後手ならB_i点得られる。

Nターン目終了時、すべてのiが選択されるが、その時点で得た得点の合計が高いほうが勝者となる。

先手勝ちor後手勝ち**or引き分け** どれになるか答えよ
### 解法メモ
A_i, B_iは「自分が取れる点数」「相手から取れる点数」と考えられ、どちらも同じ重み。

A_i+B_iで降順ソートして交互に取っていき、点数を比較すればよい。
### 実装例
```cpp
// Day4 Fun Game
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        vector<pair<int, int>> a(n);
        rep(i, n) cin >> a[i].first;
        rep(i, n) cin >> a[i].second;

        sort(all(a), [](const auto& x, const auto& y) { return x.first + x.second > y.first + y.second; });
        ll s1 = 0, s2 = 0;
        rep(i, n) {
            if (i % 2 == 0) {
                s1 += a[i].first;
            } else {
                s2 += a[i].second;
            }
        }
        if (s1 == s2) {
            cout << "Tie\n";
        } else if (s1 > s2) {
            cout << "First\n";
        } else {
            cout << "Second\n";
        }
    }
}
```

## Day 4: Powers Game
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/powers-of-two-game
### 問題概要
T<=10^6 

n(<=10^6)が与えられる。

$\*2^1\ \*2^2\ \*2^3\*\ \cdots\ \*2^n$ という列があり、各手番で＊を＋ーどちらかに変更する。

アスタリスクがなくなる、n回目の手番の終了時点で式の値が17の倍数なら後手の勝利、そうでなければ先手の勝利
### 解法メモ
mod 17で考えると$2^i$は$1,2,4,8,16,15,13,9,1,2, \cdots$と周期8でループする。

この列を見ると、1+16=2+15=4+13=8+9=17と、3つ飛ばした2つを足すと17になる。
    mod17で(1,2,4,8,-1,-2,-4,-8)と見て、符号は関係ないので4つごとに区切る方が分かりやすいかもしれない

nが8の倍数なら後手がこのペアの符号を一致させれば17の倍数になり、必勝。
    先手が1の符号を+にしたら-1の符号を+にする、4の符号を-にしたら-4の符号を-にするといった感じ。

それ以外の場合、先手が必勝になる。
- 列を8個ずつに区切り、最後の1...7個以外は↑の、ペアの符号を一致させる戦略を今度は先手側が取ることにする。
- そうすると、最後の1...7個以外は合計が0(mod 17)になる。
- 最後の1...7個は、先手が適切にペアを作れば、後手がどう頑張っても17の倍数にすることはできない
```
ラストn%8個 -> 先手が初手で選ぶものの候補
(1)       -> (1)
(1,2)     -> (1,2)
(1,2,4)   -> (1,2,4)
(1,2,4,8) -> (1,2,4,8)

(1,2,4,8,-1)        -- (1,-1)を合わせて0にする---------------> (2,4,8)
(1,2,4,8,-1,-2)     -- (1,-1),(2,-2)を合わせて0にする--------> (4,8)
(1,2,4,8,-1,-2,-4)  -- (1,-1),(2,-2)(4,-4)を合わせて0にする--> (8)
```

### 実装例
```cpp
// Day4 Powers Game
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;
    rep(_, t) {
        int n;
        cin >> n;
        cout << (n % 8 == 0 ? "Second" : "First") << "\n";
    }
}
```
## Day 5: Deforestation
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/deforestation
### 問題概要
T<=100

N(<=500)頂点の、頂点1を根とする木が与えられる。

各手番でプレイヤーは辺を1つ選び、木のそこから先の部分木を削除する。

頂点1しかない状態で手番が回ってきたら負け。
### 解法メモ
AGC017Dと全く同じ。各部分木のGrundy数を考える。
### 実装例
```cpp
// Day5 Deforestation
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;

    rep(_, t) {
        int n;
        cin >> n;
        vector<vector<int>> G(n);
        rep(i, n - 1) {
            int u, v;
            cin >> u >> v;
            u--, v--;
            G[u].push_back(v);
            G[v].push_back(u);
        }

        vector<bool> seen(n);
        auto dfs = [&](auto dfs, int u) -> int {
            seen.at(u) = true;
            int grundy = 0;
            for (const auto &e : G.at(u)) {
                if (seen.at(e)) continue;
                grundy ^= 1 + dfs(dfs, e);
            }
            return grundy;
        };

        cout << (dfs(dfs, 0) != 0 ? "Alice" : "Bob") << "\n";
    }
}
```

## Day 5: Tower Breakers - The Final Battle
https://www.hackerrank.com/contests/5-days-of-game-theory/challenges/final-tower-breakers
### 問題概要
T<=100

最初、高さN(<=10^18)の塔が1つだけあり、変数HはH=Nで初期化されている。これらを使ってゲームを行う。

先手はゲーム中支払うコインをできるだけ少なくしようとし、後手はできるだけ多くのコインを稼ごうとする。



各手番で、先手、後手の行う行動は異なり、以下を行う。
- 先手は高さHの塔を、より小さな、和がHである複数の塔の列h_1, h_2, ..., h_xに分割する。分割後の塔の数xやh_iは自由に選べる。
- 後手は、直前に先手が分割したx個の塔から塔kを選ぶ(1<=k<=x)。その後、先手は後手にコインk^2を払い、Hにh_kを代入する
先手の手番が回ってきた時点でH=1であればゲームを終了する。

### 解法メモ
難しい。

まず、たくさん分割してしまうと先手が渡さなければならないコインは$k^2$と2次関数で大きくなるうえ、そもそも適当にやっても全部2分割してしまえば$\log_2 N$程度のターン数で終わる。

分割後の個数xを大きくすることで先手が得られるメリット（次回のHが小さくなるのでトータルのターン数が少なくなる）は、デメリット（後手にこのターン支払うコインが増える）よりもかなり小さそうだという感覚になる。



とりあえず2分割しかしないものとして実験してみると、H=2の時を除けば常にk=1を選ばせるのが良さそうだと予想できる。



常に2分割が最適かというとそうではなく、3番目の要素に1つずらすことで上手くいくような場合もあるはずなので、どうするか考える必要がある。


N=tの時の最適コストを$f(t)$とすると、以下の式が満たされていればよい。
$$
f(h_1)+1^2 \leq f(h_2)+2^2 \leq f(h_3)+3^2 \leq\dots\leq f(h_x)+x^2\
$$

$f(t)$は非常にゆっくり単調増加する関数なので、$f(t)$の値が変わる境目がどこなのかを求めていって、探索範囲が$10^{18}$を超えた時点で打ち切れば、あとは境目を格納した配列を見るだけでクエリに答えることができる。

```cpp
// Day5 Tower Breakers - The Final Battle
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
#define reps(i, n) for (int i = 1, i##_len = (n); i <= i##_len; ++i)
#define all(v) begin(v), end(v)
using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    vector<ll> bound = {1, 1, 1, 1, 2};  // bound[t] := f(s)<=tとなる最大のs
    auto f = [&](ll t) -> ll {
        assert(t <= bound.back());
        return lower_bound(all(bound), t) - bound.begin();
    };
    while (bound.back() < 1'000'000'000'000'000'000) {
        ll ft = bound.size() - 1;
        ll t = bound.back();

        // x*x > ftだと、後手がこれを取るだけで良くなってしまうので、これ以降は候補にならない
        int x = int(sqrt(ft + 1));  // 分割する数
        ll sum = 0;
        reps(i, x) { sum += bound[ft + 1 - i * i]; }
        bound.push_back(sum);
    }
    int t;
    cin >> t;
    rep(_, t) {
        ll h;
        cin >> h;
        cout << f(h) << "\n";
    }
}
```
