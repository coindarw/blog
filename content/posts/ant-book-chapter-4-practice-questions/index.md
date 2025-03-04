+++
title = '蟻本 上級編 練習問題 解法メモ集'
date = 2025-03-04T14:19:42+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

蟻本の「練習問題」のところの問題を埋めました。
問題は面白いものがそろっているのですが、ほとんどが POJ の問題であり、

- 実行が遅く、TL が厳しい
- コンパイラが非常に古い（C++98）
- 一部の制約が書いていないため、エスパーする必要があることがある

などとつらい要素が多いです。本を読んだ後に練習したい場合、[AtCoder 版！蟻本 (上級編)](https://qiita.com/drken/items/9b311d553aa434bb26e4)をやったりする方が良いと思います。

ただ、以下のように色々使える手はあって、埋めようと思うと意外と何とかなる印象です。

- POJ 以外のジャッジに同じ問題があるならそちらに提出する
  - 半分程度の問題は他のジャッジに存在する
- ChatGPT に古いバージョンの C++の書き方に直してもらう
- clangd のようなツールを使って CE になりそうなところを指摘してもらう

<!--more-->

https://vjudge.net/problem でタイトルを検索すると他のジャッジが使える場合も多いです。いろいろなアカウントを作るのは面倒なので、vjudge から bot 提出するのが多分楽です。
ただし、Baekjoon(BOJ)のように vjudge の bot をサポートしていなく、自分のアカウントを作る必要のある OJ もあります。
なお、Baekjoon(BOJ)には既に本家には提出できない GCJ の問題があり、蟻本の GCJ の章の問題を解くことができます。

基本的には蟻本の対応する部分は読んでいる前提で、アルゴリズムの説明などは入れていません

なお、記事はObsidian(Markdownで書けるノートアプリ的なもの)から適当なPythonスクリプトで変換しているため、一部表示がおかしくなっている可能性があります。

## 4-1 より複雑な数学的問題

- mod の世界
  - [POJ 1150 The Last Non-zero Digit(UVa 10212)](../practice/poj-1150)
  - [POJ 1284 Primitive Roots](../practice/poj-1284)
  - [POJ 2115 C Looooops(BOJ 6670, Gym 100811C)](../practice/poj-2115)
  - [POJ 3708 Recurrent Function](../practice/poj-3708)
  - [POJ 2720 Last Digits](../practice/poj-2720)
  - [GCJ Japan 2011 決勝 B バクテリアの増殖(BOJ 12445, 12446)](../practice/gcj-japan)
- 行列
  - [POJ 2345 Central heating(EOlymp 219)](../practice/poj-2345)
  - [POJ 3532 Resistance](../practice/poj-3532)
  - [POJ 3526 The Teacher's Side of Math(AOJ 1284, BOJ 3904, Gym 101415J, UVa 1397)](../practice/poj-3526)
- 数え上げ
  - [POJ 2407 Relatives(UVa 10299)](../practice/poj-2407)
  - [POJ 1286 Necklace of Beads(BOJ 9817, EOlymp 2227)](../practice/poj-1286)
  - [POJ 2409 Let it Bead(BOJ 6567)](../practice/poj-2409)
  - [AOJ 2164 Revenge of the Round Table(BOJ 22623)](../practice/aoj-2164)
  - [AOJ 2214 Warp Hall](../practice/aoj-2214)

## 4-2 ゲームの必勝法を編み出せ！

- 考察・動的計画法等
  - [POJ 1082 Calendar Game(UVa 1557)](../practice/poj-1082)
  - [POJ 2068 Nim(AOJ 1230, BOJ 9157)](../practice/poj-2068)
  - [POJ 3688 Cheat in the Game](../practice/poj-3688)
  - [POJ 1740 A New Stone Game](../practice/poj-1740)
- Nim・Grundy 数
  - [POJ 2975 Nim(BOJ 7685)](../practice/poj-2975)
  - [POJ 3537 Crosses and Crosses(Gym 100078C)](../practice/poj-3537)
  - [Codeforces 138D World of Darkraft](../practice/codeforces-138d)
  - [POJ 2315 Football Game(出題ミス？)](../practice/poj-2315)

## 4-3 グラフマスターへの道

- 強連結成分分解
  - [POJ 3180 The Cow Prom(luogu P2863)](../practice/poj-3180)
  - [POJ 1236 Network of Schools(luogu P2746)](../practice/poj-1236)
- 2-SAT
  - [POJ 3678 Katu Puzzle](../practice/poj-3678)
  - [POJ 2723 Get Luffy Out](../practice/poj-2723)
  - [POJ 2749 Building roads](../practice/poj-2749)
- LCA
  - [POJ 1986 Distance Queries](../practice/poj-1986)
  - [POJ 3728 The merchant](../practice/poj-3728)

## 4-4 厳選！ 頻出テクニック（2）

- スタック
  - [POJ 3250 Bad Hair Day(luogu P2866)](../practice/poj-3250)
  - [POJ 2082 Terrible Sets](../practice/poj-2082)
  - [POJ 3494 Largest Submatrix of All 1’s](../practice/poj-3494)
- デック
  - [POJ 2823 Sliding Window](../practice/poj-2823)
  - [POJ 3260 The Fewest Coins(BOJ 6205,Luogu P2851)](../practice/poj-3260)
  - [POJ 1180 Batch Scheduling(BOJ 5498, EOlymp 4144, DMOJ ioi02p4)](../practice/poj-1180)
  - [AOJ 1070 FIMO sequence](../practice/aoj-1070)

## 4-5 工夫を凝らして賢く探索

- 枝刈り
  - [POJ 1011 Sticks](../practice/poj-1011)
  - [POJ 2046 Gap(BOJ 3937)](../practice/poj-2046)
  - [POJ 3134 Power Calculus(AOJ 1271)](../practice/poj-3134)
- A*, IDA*
  - [POJ 3523 The Morning after Halloween(AOJ 1281,BOJ 3901,Gym 101415G)](../practice/poj-3523)
  - [POJ 2032 Square Carpets(AOJ 1128,BOJ 22814)](../practice/poj-2032)
  - [UVA 10181 15-Puzzle Problem](../practice/uva-10181)

## 4-6 分けて解いてまとめる！“分割統治法”

- 列の分割統治
  - [POJ 1854 Evil Straw Warts Live(BOJ 4309,UVA 10716)](../practice/poj-1854)
- 平面の分割統治
  - [GCJ 2009 World Finals B Min Perimeter(BOJ 12611,12612,Gym 100240K)](../practice/gcj-2009)
  - [Codeforces 97B Superset](../practice/codeforces-97b)
- ツリーの重心分解
  - [POJ 2114 Boatherds(BOJ 6669,Gym 100811B)](../practice/poj-2114)
  - [UVa 12161 Ironman Race in Treeland](../practice/uva-12161)
  - [SPOJ QTREE5 Query on a tree V](../practice/spoj-qtree5)

## 4-7 文字列を華麗に扱う

- 動的計画法
  - [AOJ 2212 Stolen Jewel](../practice/aoj-2212)
  - [Codeforces 86C Genetic Engineering](../practice/codeforces-86c)
- 文字列検索等
  - [Codeforces 25E Test](../practice/codeforces-25e)
  - [AOJ 1312 Where's Wally](../practice/aoj-1312)
- 接尾辞配列
  - [POJ 1509 Glass Beads(BOJ 3492,EOlymp 2912,SPOJ BEADS)](../practice/poj-1509)
  - [POJ 3415 Common Substrings](../practice/poj-3415)
  - [POJ 3729 Facer's string](../practice/poj-3729)
  - [AOJ 2292 Common Palindromes(BOJ 22496)](../practice/aoj-2292)
  - [Codeforces 123D String](../practice/codeforces-123d)
