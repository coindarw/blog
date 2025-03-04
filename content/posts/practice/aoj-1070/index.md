+++
title = "AOJ 1070 FIMO sequence"
date = 2025-03-04T13:52:54+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=1070&lang=ja

https://vjudge.net/problem/Aizu-1070
<!--more-->
## 問題概要
- 添え字は1-indexed
- 数列前半とは数列の長さを$N$として、1番目から$\left\lceil \dfrac n2\right\rceil$番目までを指す。$N=0$なら空。
- 数列後半とは数列のうち、前半以外を指す。$N=0,1$なら空。

- 最初、数列$A$は空である。以下のクエリに答えよ。
	- クエリ$(0,x)$：数列の末尾に$x$を挿入する。このクエリによって与えられる$x$は重複しない
	- クエリ$(1)$：数列の長さを$N$として、$\left\lceil \dfrac n2\right\rceil$番目の要素を削除する（前半部分の末尾を削除）
	- クエリ$(2)$：数列前半の最小値を求める
	- クエリ$(3)$：数列後半の最小値を求める
	- クエリ$(4,i)$：現在の数列にクエリ$(2)$と$(1)$を順に繰り返したときの出力のうち、現在前半に含まれるものの中で、$i$番目に小さいものを出力（実際には要素は削除しない）
	- クエリ$(5,i)$：現在の数列にクエリ$(3)$と$(1)$を順に繰り返したときの出力のうち、現在前半に含まれるものの中で、$i$番目に小さいものを出力（〃）
	- クエリ$(6)$：数列前半の最大値を求める
	- クエリ$(7)$：数列後半の最大値を求める
	- クエリ$(8,i)$：現在の数列にクエリ$(6)$と$(1)$を順に繰り返したときの出力のうち、現在前半に含まれるものの中で、$i$番目に大きいものを出力（〃）
	- クエリ$(9,i)$：現在の数列にクエリ$(7)$と$(1)$を順に繰り返したときの出力のうち、現在前半に含まれるものの中で、$i$番目に大きいものを出力（〃）
### 制約
- マルチテストケース
- データ全体でのクエリーの数の和は$2\times 10^5$を超えない
- クエリの結果数列の長さは$2\times10^4$を超えない
- 最小値・最大値を考える対象が空になるような入力が来ることはない
## 解法メモ
minについてだけ解ければmaxは同じなので、クエリ1~5について考える
### クエリ2 前半のmin
- クエリ2は前半部分のminを管理すればよく、数列に対して以下のクエリが処理できればよい
	- 末尾への追加（中央の要素を削除すると後半の最初の要素が前半にくる）
	- 末尾の削除
	- 数列全体のminの取得

- これはstackに要素xをpushする代わりに$\min(x, stackの末尾要素)$をpushするようにすれば、末尾要素が答えになる。

- [SWAG(Sliding Window Aggregation)再考 - Motsu_xe 競プロのhogeやfuga (hatenablog.com)](https://motsu-xe.hatenablog.com/entry/2021/05/13/224016) の記事に書かれている「SWAGのstack版」がこれ
### クエリ3 後半のmin
- クエリ3に答えるには以下のクエリが処理できればよく、これはスライド最小値やSWAGと呼ばれるデータ構造が使える
- 後のクエリで役に立つのでスライド最小値で実装するのがよい
	- 末尾への追加
	- 先頭の削除
	- 数列全体のminの取得
### クエリ4
- 最小値になることがある要素をBITやpbdsのtree等で管理できれば$k$番目の要素がわかる
- このようなことができるかどうか考える


- 前半に含まれる$A_i$は、削除操作を続けるとどこかで前半部分が$(A_1, A_2, \cdots, A_i)$になるタイミングが存在する。
- そのタイミング以前の前半部分はこれの後ろにいくつかの要素が連なる形になる


- 自分が末尾要素になったタイミングで最小であれば最小値として出力され、そうでなければ最小値になることはない


- 自分より前の要素と比較して最小であれば出力に出てくる
クエリ2のstackで自分がminになる要素を保存しておけばよい


- クエリ0(挿入)、1(中央削除)による差分は$O(1)$でわかる
### クエリ5
- 後半部分の各要素について、自分が以降の要素で最小値になるようなものの中で$i$番目のものが分かればよい


- これはスライド最小値で管理するデータそのものなので、クエリ3をスライド最小値で実装し、クエリ5ではdequeの中の該当する要素を出力すればよい
## 実装例
```cpp
#include <bits/stdc++.h>
using ll = long long;
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
constexpr int inf = 2000'000'000;
#define all(v) begin(v), end(v)
using namespace std;

template <typename T>
struct BIT {
    int n;
    vector<T> data;
    BIT(int n) : n(n), data(n + 1, 0) {}
    BIT() {}
    void add(int i, T x = 1) {
        for (i++; i <= n; i += (i & -i)) data[i] += x;
    }
    T sum(int i) {
        T s(0);
        for (i++; i; i -= (i & -i)) s += data[i];
        return s;
    }
    T sum(int l, int r) { return sum(r - 1) - sum(l - 1); }
    T get(int i) { return sum(i) - sum(i - 1); }

    int lower_bound(T w) {
        if (w <= 0) {
            return 0;
        }
        int x = 0, r = 1;
        while (r < n) r <<= 1;
        for (int len = r; len > 0; len >>= 1) {
            if (x + len <= n && data[x + len] < w) {
                w -= data[x + len];
                x += len;
            }
        }
        return x;
    }
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    while (true) {
        int q;
        cin >> q;
        if (q == 0) break;

        // クエリ先読み
        vector<pair<int, int>> query(q);
        rep(i, q) {
            int c, x = -1;
            cin >> c;
            if (c == 0 || c == 4 || c == 5 || c == 8 || c == 9) cin >> x;
            query[i] = {c, x};
        }

        // 座圧
        vector<int> vx;
        rep(i, q) {
            if (query[i].first == 0) vx.push_back(query[i].second);
        }
        sort(all(vx));
        rep(i, q) {
            if (query[i].first == 0) {
                query[i].second = lower_bound(all(vx), query[i].second) - vx.begin();
            }
        }

        // 配列などの宣言
        deque<int> head, tail;
        vector<int> head_min_stack, head_max_stack;
        head_min_stack.push_back(inf);
        head_max_stack.push_back(-inf);

        // bit_min: クエリ4で出力されうる要素の集合
        BIT<int> bit_min(vx.size()), bit_max(vx.size());

        deque<int> tail_slide_min, tail_slide_max;  // 後半のスライド最小値/最大値

        // 追加・削除によって後半の先頭要素が前半の末尾に移動する際の処理
        auto move_ = [&]() {
            head.push_back(tail.front());
            tail.pop_front();

            // スライド最小値/最大値からpop
            if (head.back() == tail_slide_min.front()) tail_slide_min.pop_front();
            if (head.back() == tail_slide_max.front()) tail_slide_max.pop_front();

            // 前半のmin/maxと、クエリ4,8で出てくる値の前半部分を追加
            if (head.back() < head_min_stack.back()) {
                head_min_stack.push_back(head.back());
                bit_min.add(head.back());
            } else {
                head_min_stack.push_back(head_min_stack.back());
            }
            if (head.back() > head_max_stack.back()) {
                head_max_stack.push_back(head.back());
                bit_max.add(head.back());
            } else {
                head_max_stack.push_back(head_max_stack.back());
            }
        };

        // クエリ0: 数列末尾に要素を追加
        auto push_tail_back = [&](int x) {
            tail.push_back(x);

            // 後半のスライド最小値/最大値の更新
            while (!tail_slide_min.empty() && tail_slide_min.back() > x) tail_slide_min.pop_back();
            tail_slide_min.push_back(x);
            while (!tail_slide_max.empty() && tail_slide_max.back() < x) tail_slide_max.pop_back();
            tail_slide_max.push_back(x);

            if (head.size() < tail.size()) move_();
        };

        // クエリ1: 数列中央の要素の削除
        auto pop_head_back = [&]() {
            int erased = head.back();
            head.pop_back();
            head_min_stack.pop_back();
            head_max_stack.pop_back();
            if (bit_min.get(erased) == 1) bit_min.add(erased, -1);
            if (bit_max.get(erased) == 1) bit_max.add(erased, -1);

            if (head.size() < tail.size()) move_();
            return erased;
        };

        rep(i, q) {
            auto [c, x] = query[i];
            if (c == 0) {  // push back x;
                push_tail_back(x);
            } else if (c == 1) {  // pop head back
                int erased = pop_head_back();
                cout << vx[erased] << "\n";
            } else if (c == 2) {  // head min
                cout << vx[head_min_stack.back()] << "\n";
            } else if (c == 3) {  // tail min
                cout << vx[tail_slide_min.front()] << "\n";
            } else if (c == 4) {
                cout << vx[bit_min.lower_bound(x)] << "\n";
            } else if (c == 5) {
                cout << vx[tail_slide_min[x - 1]] << "\n";
            } else if (c == 6) {  // head max
                cout << vx[head_max_stack.back()] << "\n";
            } else if (c == 7) {  // tail max
                cout << vx[tail_slide_max.front()] << "\n";
            } else if (c == 8) {
                cout << vx[bit_max.lower_bound(bit_max.sum(0, vx.size()) - x + 1)] << "\n";
            } else if (c == 9) {
                cout << vx[tail_slide_max[x - 1]] << "\n";
            }
        }
        cout << "end\n";
    }
}
```
