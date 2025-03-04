+++
title = "POJ 2315 Football Game (出題ミス？)"
date = 2025-03-04T13:53:40+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

Shttp://poj.org/problem?id=2315

https://vjudge.net/problem/POJ-2315

まず問題文が分かりづらいです。勝利条件がどうしても読解できなかったので検索しましたが、どうしてこうなるのかわかりません

**また、後述するようにAC自体は取っていますが反例があり、出題ミスではないかと考えています**
<!--more-->
## 問題概要
- AliceとBobがゲームを行う。Aliceから順に交互に手番が回る。
- サッカーゴールの前に$N$個のボールがあり、各ボールのゴールまでの距離は$S_i$
- 各手番では(1個以上)$M$個以下のボールを選び、キックしてゴールに近づける。全てのボールがゴールした状態で手番が回ってきた方の負け
- ボールの半径は$R$。ボールは重心の位置がずれているので周りの長さ（$2\pi R$）の整数倍しか転がらない。AliceとBobはボールをキックする際、$L$以下の任意の距離にコントロールできる
### 制約
- $1\leq M\leq N\leq30$
- $0<L<10^8$
- $0<R<10^4$
- $0<S_i<10^8$
## メモ
- 各山に$A_i=\left\lceil \dfrac{S_i}{2\pi R}\right\rceil$個の石があり、各ターンで1個以上$M$個以下の山を選び、それぞれから$r=\left\lfloor\dfrac{L}{2\pi R}\right\rfloor$個まで取ることができるNimとみなせる

- https://en.wikipedia.org/wiki/Nim#Index-k_nim
- 英語版Wikipediaにあるが、これはIndex-K Nimと呼ばれるゲームで、個数制限がない場合は各$A_i$を2進表記してビットごとの個数をカウントした時、それぞれの個数が全て$M+1$の倍数であれば後手の勝ち、そうでなければ先手の勝ちとなる。
- （NimKと呼ぶ場合もあるが、調べた感じ「山を1つ選び、$K$個以下の石を取れるNim」、「山を$K$個以下選び、それぞれの山から好きな数石を取れるNim」どちらもNimKと呼んでいる例がある）

- 個数制限がある場合も同じ項に載っており、$B_i=A_i\%(r+1)$として、数列$B$に対してNimKと同じ判定を行えばよいらしい。これでAC**は**取れる。

## 出題ミス？
- ただ、愚直コードを走らせたときに$r=3,M=2$の時、いくつかの盤面の勝ち負けが以下のようになった
- $\bmod 4$だと同じになる数列の勝敗が変わってしまうので、$B_i=A_i\%(r+1)$としてはいけないはず

- Wikipediaに載っているE. H. Mooreの文献を読んでみたが取れる数に制限のあるものについては載っていないので、どこ由来の情報なのか不思議
	- NimKと呼ばれるゲームが2種類あることに由来する誤解ではないかと推測

```
1 1  2 0 : 先手勝ち
1 1  2 4 : 先手勝ち
1 1  6 0 : 先手勝ち
1 1  6 4 : 先手負け
1 1 10 4 : 先手負け
1 1 10 8 : 先手負け
1 5 10 8 : 先手負け
5 5 10 8 : 先手負け
5 9  6 8 : 先手負け

1 1 1 4 : 先手勝ち
1 1 1 0 : 先手負け
```


http://poj.org/showcontest?contest_id=1131 Editorialなどもなさそう
## 実装例
```cpp
#define _USE_MATH_DEFINES
#include <cmath>
#include <iostream>
#include <vector>
#define rep(i, n) for (int i = 0; i < (int)(n); i++)
using namespace std;

bool nimK(const vector<int> &v, int k) {
    for (int b = 0; b < 30; ++b) {
        int cnt = 0;
        for (int i = 0; i < (int)v.size(); ++i) {
            if (v[i] >> b & 1) cnt++;
        }
        if (cnt % (k + 1) != 0) return true;
    }
    return false;
}

int main() {
    int n, m, l, r;
    while (cin >> n >> m >> l >> r) {
        vector<int> s(n);
        rep(i, n) cin >> s[i];

        int k = (int)floor(l / (2.0 * M_PI * r));

        vector<int> a(n);
        rep(i, n) a[i] = (int)ceil(s[i] / (2.0 * M_PI * r));

        vector<int> b(n);
        rep(i, n) b[i] = a[i] % (k + 1);

        if (nimK(b, m)) {
            cout << "Alice" << endl;
        } else {
            cout << "Bob" << endl;
        }
    }
}
```


## 反例について
愚直コード
$M=2, N=3,4$の列について1(勝ち),0(負け)を出力する
```cpp
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0; i < (int)(n); i++)
#define reps(i, n) for (int i = 1; i <= (int)(n); i++)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
#define all(v) begin(v), end(v)
using namespace std;

int main() {
    int r = 3;

    map<vector<int>, bool> mp;

    auto analysis = [&](auto f, vector<int> a) -> bool {
        int n = a.size();
        sort(all(a));
        if (mp.contains(a)) return mp[a];
        if (all_of(all(a), [](int x) { return x == 0; })) return false;

        rep(i, n) reps(ic, min(r, a[i])) {  // 2つの山を選ぶ場合
            rep2(j, i + 1, n) reps(jc, min(r, a[j])) {
                auto b = a;
                b[i] -= ic;
                b[j] -= jc;
                if (!f(f, b)) {
                    return mp[a] = true;
                }
            }
        }
        rep(i, n) reps(ic, min(r, a[i])) {  // 1つの山を選ぶ場合
            auto b = a;
            b[i] -= ic;
            if (!f(f, b)) {
                return mp[a] = true;
            }
        }
        return mp[a] = false;
    };

    vector<int> vec;
    auto rec = [&](auto rec, int len) {  // len要素、0~9からなる広義単調増加列を列挙する
        if (vec.size() == len) {
            int ret = analysis(analysis, vec);
            rep(i, vec.size()) cout << vec[i];
            cout << " : " << ret << endl;
            return;
        }
        rep(i, 10) {
            if (vec.size() && vec.back() > i) continue;
            vec.push_back(i);
            rec(rec, len);
            vec.pop_back();
        }
    };
    rec(rec, 3);
    rec(rec, 4);
    return 0;
}
```

$M=2,r=3$での$A=\{1, 1, 4, 6\}$からの遷移
ここから遷移できる以下のパターンは全て勝ち盤面なので$A$は負け盤面になるはず
```
0046 : 0044にできるので勝ち

0136 : 0115〃
0126 : 0115〃
0116 : 0115〃

0145 : 0115〃
0144 : 0044〃
0143 : 0111〃

1135 : 1105〃
1134 : 0111〃
1133 : 1110〃

1125 : 1105〃
1124 : 1101〃
1123 : 1101〃

1115 : 0115〃
1114 : 0111〃
1113 : 0111〃

0146 : 0115〃

1136 : 1105〃
1126 : 1105〃
1116 : 1105〃

1145 : 0115〃
1144 : 0044〃
1143 : 1110〃
```