+++
title = 'priority_queueで今選べる区間を貪欲に選ぶ問題とその個人的にバグらせづらい実装'
date = 2023-10-23T02:17:48+09:00
tags = ['優先度付きキュー', '貪欲法', 'イベントソート', '競技プログラミング']
+++

これ系の問題ちょいちょい見るのに解くとき毎回記憶喪失になるので備忘録

新規性はありません

変数nowとitrを進めたりするような実装の方がいいんだろうけどバグらせるのでイベントソートっぽく解きます
（尺取り法とかこういうのを共通して簡単に扱える方法はないのだろうか）

<!--more-->

## 問題概要

次のような問題

- $N$個の仕事があり、それぞれ$L_i$日目～$R_i$日目の間でやる必要がある。同じ日に複数の仕事を行うことはできない。最大でいくつの仕事を行うことができるか？
    - 制約 $N\leq 2\mathrm e5, 1\leq L_i,R_i\leq 1\mathrm e18$

[ABC214E](https://atcoder.jp/contests/abc214/tasks/abc214_e), [ABC325D](https://atcoder.jp/contests/abc325/tasks/abc325_d)などが該当する。問題文や制約はいろいろ考えられるがこの記事では上のもので行く。

- 無限個の箱があり、$N$個のボールを箱に入れていく。箱には1個までしかボールは入らず、また$i$番目のボールは$L_i$以上$R_i$以下の番号のついた箱に入れる必要がある。全てのボールを箱にしまえるか？（ABC214E）
- $N$個の商品があり、各商品に印字していく。商品$i$は時刻$T_i$から$D_i$単位時間の間（境界時刻含む）印字することが可能。同時に1つの商品にしか印字できず、また一度印字したら1単位時間は印字機が使えない。最大いくつの商品に印字できるか？（ABC325D）

## 解き方と実装

一見区間スケジューリング問題っぽい見た目をしているが、区間[1,3]と[2,2]があったときに、区間スケジューリング問題なら[2,2]を優先した方がいいのに対し、今回の問題だと[1,3]を優先したほうがいい。同じ戦略は使えない。

これは次のように解ける

- 考えられる日付を小さいほうから順に列挙していき、今選べる仕事の中で締め切りが最も早いものから順にやる

実装方針はいろいろ考えられるが、

「今何日目かを表す変数nowとソート済みの区間の今何番目かを示す変数itrを持っておき、itr番目の仕事を選ぶならnow++する。今考えているLiがnowよりも後ならnow=L[i]にする。」

のような感じで実装することが多そう

ただ、この方法だと変数nowを管理するのが結構面倒

priority_queueを使ったイベントソート的な感じで実装するのが楽だと思っている（メモリは少し余計に使うが）

以下の2つのイベントを考え、

1. $L$日目～$R$日目の間で行う仕事を候補に追加
2. まだ他の仕事が入っていなければ$L$日目に仕事を行う

上手くスケジュールを決めると、仕事を行う日付の候補は「$L_i$日目」か「仕事を行った日の翌日」としてよい

イベント2($L$日目)を処理して仕事を行うことを決めたら、イベントを管理するpriority_queueに($L+1$日目)をpushすればOK

優先度をイベント1>イベント2にすることに注意

イベント2がpushされる回数は高々$2N$回なので計算量は$O(N\log N)$で元の解法と変わらない

### 実装例

```cpp

constexpr ll linf = 4'000'000'000'000'000'000;

auto solve = [](const vector<pair<ll, ll>> v) -> int {
    // v: 区間[l, r]
    using P = pair<ll, ll>;
    priority_queue<P, vector<P>, greater<P>> events;         // イベントソート
    priority_queue<ll, vector<ll>, greater<ll>> candidates;  // 現時点で使えるRiの候補
    unordered_set<ll> usedTime(v.size());                    // 使用済みの日付
    // L日目 or 既に使用した日付+1 しか候補にならない
    for (auto [l, r] : v) {
        assert(l <= r && r < linf);
        events.push({l, r});     // 仕事の追加[l, r] イベント1
        events.push({l, linf});  // L日目時点で可能なら仕事を行う イベント2
    }
    int ans = 0;
    while (!events.empty()) {
        auto [l, r] = events.top();
        events.pop();
        if (r != linf) {  // 仕事の追加 イベント1
            candidates.push(r);
        } else if (!usedTime.contains(l)) {                                        // L日目時点で可能なら仕事を行う イベント2
            while (!candidates.empty() && candidates.top() < l) candidates.pop();  // できないことが確定した仕事を削除
            if (!candidates.empty()) {                                             // できる仕事が存在するならする
                candidates.pop();
                ans++;
                usedTime.insert(l);
                events.push({l + 1, linf});  // 翌日も仕事ができるなら行う
            }
        }
    }
    return ans;
};
```

## 問題リンク

- [https://atcoder.jp/contests/abc325/tasks/abc325_d](https://atcoder.jp/contests/abc325/tasks/abc325_d)
- [https://atcoder.jp/contests/abc214/tasks/abc214_e](https://atcoder.jp/contests/abc214/tasks/abc214_e)