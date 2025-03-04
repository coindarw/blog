+++
title = "POJ 1082 Calendar Game(UVa 1557)"
date = 2025-03-04T13:53:14+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=1082
https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=24&page=show_problem&problem=4332

https://vjudge.net/problem/POJ-1082
https://vjudge.net/problem/UVA-1557
<!--more-->
## 問題概要
- AdamとEveは1900年から2001年までのカレンダーを使ってゲームを行う。
- 最初、カレンダー上のある日付に駒が1つだけおいてあり，Adam→Eve→Adam→Eve→...と交互に駒を進めていく。
- 自分のターンで2001年11月4日以降の日付に駒を進めると勝利する。
- 駒は自分の手番に翌日、もしくは翌月の同じ日に移動させることができ、翌月同じ日が存在しない場合は翌日にのみ移動させることができる。
- AdamがEveに勝利する戦略が存在するか判定せよ
### 入力
- テストケース数、最初の日付

## 解法メモ
- ざっくり100年×365日=36500日なので、素直にメモ化再帰するのが楽そう

- Javaでも組んでみたが、`java.util.Calendar` を使うとMLEした

- なお、数学的に考えてかなり短く実装することもできるよう
	- https://debiru.hatenablog.com/entry/20220213/poj1082
## 実装例
C++で適当に作った日付構造体を使った。
MLがきついので標準ライブラリのカレンダークラスなどは辛そう。
	`struct Date { long long y, m, d; };` などとするとそれだけでMLEする

```cpp
#include <cstdio>
#include <iostream>
#include <map>
using namespace std;

const int DAYS[] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
struct Date {
    int y, m, d;
    Date(int y, int m, int d) : y(y), m(m), d(d) {}
    bool operator<(const Date& rhs) const {
        if (y != rhs.y) return y < rhs.y;
        if (m != rhs.m) return m < rhs.m;
        return d < rhs.d;
    }
    bool operator==(const Date& rhs) const { return y == rhs.y && m == rhs.m && d == rhs.d; }
    bool is_leap() { return y % 4 == 0 && (y % 100 != 0 || y % 400 == 0); }
    bool is_valid() { return y != -1; }
    Date next_day() {
        Date res = *this;
        res.d++;
        if (res.m == 2 && res.is_leap() && res.d == 29) return res;
        if (res.d > DAYS[res.m]) {
            res.d = 1;
            res.m++;
            if (res.m > 12) {
                res.m = 1;
                res.y++;
            }
        }
        return res;
    }
    Date next_month_day() {
        Date res = *this;
        res.m++;
        if (res.m == 2 && res.is_leap() && res.d == 29) return res;
        if (res.m > 12) {
            res.m = 1;
            res.y++;
        }
        if (res.d > DAYS[res.m]) return Date(-1, -1, -1);
        return res;
    }
};

map<Date, bool> memo;
Date LAST_DATE = Date(2001, 11, 4);

bool solve(Date date) {
    if (date == LAST_DATE) return false;
    if (!(date < LAST_DATE)) return true;
    if (memo.count(date)) return memo[date];

    Date nxt1 = date.next_day();
    Date nxt2 = date.next_month_day();
    if (!solve(nxt1)) return memo[date] = true;
    if (nxt2.is_valid() && !solve(nxt2)) return memo[date] = true;
    return memo[date] = false;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int t;
    cin >> t;

    while (t--) {
        int y, m, d;
        cin >> y >> m >> d;
        cout << (solve(Date(y, m, d)) ? "YES" : "NO") << '\n';
    }
}
```