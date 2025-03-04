+++
title = "POJ 3708 Recurrent Function"
date = 2025-03-04T13:54:17+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

http://poj.org/problem?id=3708

https://vjudge.net/problem/POJ-3708
<!--more-->
## 問題概要

- ある関数$f(x)$があり、以下のように定義される
$$
\begin{align} f(j) &= a_j & \text{for } 1 \leq j < d, \\ f(d \times n + j) &= d \times f(n) + b_j & \text{for } 0 \leq j < d \text{ and } n \geq 1, \end{align}
$$

- ここで、集合 ${a_i}$ は ${1, 2, \ldots, d-1}$ から選ばれ、集合 ${b_i}$ は ${0, 1, \ldots, d-1}$ から選ばれる

- 以下を定義する
$$
f_x(m) = f(f(f(\cdots f(m)))) \quad x\text{ times}

$$

- 正の整数 $m$ と $k$ が与えられる。$f_x(m) = k$ となる最小の非負整数 $x$ が存在するか、存在する場合はその値を求めよ
- なお、答えが $2^{63}$ 未満であることが保証される

### 入力
マルチテストケースで、各ケースは以下のようになっている。整数-1のみの行が与えられたとき入力終了

```
d
a[1] a[2] ... a[d-1]
b[0] b[1] b[2] ... b[d-1]
m
k
```

### 制約
- $2 \leq d \leq 100$
- $0 \leq m \leq 10^{100}$
- $0 \leq k \leq 10^{100}$

## 解法メモ
かなり難しかった。
### f(x)の定義について
まず$f(x)$の定義が分かりづらい。再帰関数で書くと少しわかりやすくなる。
```python
def f(x):
    if x < d:
        return a[x]
    return f(x // d) * d + b[x % d]
```

つまり、$f(x)$は以下のような処理を行っていると考えられる。
```python
def f(x):
    xl = xをd進表記したリスト
    for i in range(len(xl)):
		if i == 0: # 最上位桁だけは順列aで置換
	  	  xl[i] = a[xl[i]]
		else: # それ以外の桁は順列bで置換
	    	xl[i] = b[xl[i]]
	return to_int(xl)
```

問題文を言い換えると以下のようになる。なお、整数$n$を$d$進表記した時の$i$桁目を$n[i]$とする。(最上位桁が$n[1]$)
> $1,\cdots,d-1$の順列$a$, $0,\cdots,d-1$の順列$b$が与えられる。
> 関数$f(x)$は以下のような関数である。
>
	$x$を$d$進表記した時、$f(x)$は$x$と同じ桁数の整数を返し、その時の$i$桁目$f(x)[i]$は以下のようになる。
> $$
		\begin{align}f(x)[i]
		=\left\{\begin{array}{ll}
		a_{x[i]}&\text{if }i=1\\
		b_{x[i]}&\text{otherwise}
		\end{array}	\right.
		\end{align}
	$$
> $m$に何回$f$を適用すると$k$になるか求めよ

### ここから解法
- $a, b$は順列なので、各桁の変換は有向閉路上の移動とみなすことができる。
- 各$k[i], m[i]$が同じ閉路上にあれば、それらが一致する$f$の適用回数は閉路のサイズを$c$として$ct+k\quad(t\geq1)$のような形で表すことができる。

- このような式が桁数分並び、それらすべてに当てはまる適用回数を求めればいい。これはGarnerのアルゴリズム（ACLのCRT）で求めることができる。
## 実装例
最初の$d$進に変換するところで多倍長整数がないと面倒なのでJavaで通した
```java
import java.util.Scanner;
import java.util.ArrayList;
import java.math.BigInteger;
import java.util.Collections;

public class Main {

    private static long[] inv_gcd(long a, long b) { // 拡張ユークリッド互除法
        a %= b;
        if (a < b)
            a += b;
        if (a == 0) {
            return new long[] { b, 0 };
        }

        long s = b, t = a;
        long m0 = 0, m1 = 1;

        while (t > 0) {
            long u = s / t;
            s -= t * u;
            m0 -= m1 * u;
            long tmp = s;
            s = t;
            t = tmp;
            tmp = m0;
            m0 = m1;
            m1 = tmp;
        }
        if (m0 < 0)
            m0 += b / s;
        return new long[] { s, m0 };
    }

    private static long[] crt(long[] r, long[] m) { // ほぼACLと同じ
        int n = r.length;
        long r0 = 0, m0 = 1;
        for (int i = 0; i < n; i++) {
            long r1 = r[i] % m[i];
            if (r1 < 0)
                r1 += m[i];
            long m1 = m[i];
            if (m0 < m1) {
                long tmp = r0;
                r0 = r1;
                r1 = tmp;
                tmp = m0;
                m0 = m1;
                m1 = tmp;
            }
            if (m0 % m1 == 0) {
                if (r0 % m1 != r1)
                    return new long[] { 0, 0 };
                continue;
            }
            long[] inv = inv_gcd(m0, m1);
            long g = inv[0], im = inv[1];

            long u1 = (m1 / g);
            if ((r1 - r0) % g != 0)
                return new long[] { 0, 0 };

            long x = (r1 - r0) / g % u1 * im % u1;

            r0 += x * m0;
            m0 *= u1;
            if (r0 < 0)
                r0 += m0;
        }
        return new long[] { r0, m0 };
    }

    private static long solve(int d, ArrayList<Integer> a, ArrayList<Integer> b, BigInteger m, BigInteger k) {
        // m,kをd進数に変換
        ArrayList<Integer> ml = new ArrayList<Integer>();
        ArrayList<Integer> kl = new ArrayList<Integer>();
        while (m.compareTo(BigInteger.ZERO) > 0) {
            ml.add(m.mod(BigInteger.valueOf(d)).intValue());
            m = m.divide(BigInteger.valueOf(d));
        }
        Collections.reverse(ml);
        while (k.compareTo(BigInteger.ZERO) > 0) {
            kl.add(k.mod(BigInteger.valueOf(d)).intValue());
            k = k.divide(BigInteger.valueOf(d));
        }
        Collections.reverse(kl);

        // 桁数が異なる場合は不可能
        if (ml.size() != kl.size()) {
            return -1;
        }

        // 1桁目のループ周期を求める。 x0*t+y0回fを適用すると1桁目がkと一致する
        ArrayList<Boolean> used0 = new ArrayList<Boolean>(d);
        for (int i = 0; i < d; i++)
            used0.add(false);
        int x0 = 0, y0 = -1;
        int ml0 = ml.get(0);
        while (true) {
            if (ml0 == kl.get(0) && y0 == -1)
                y0 = x0;
            ml0 = a.get(ml0);
            if (used0.get(ml0)) {
                break;
            }
            x0++;
            used0.set(ml0, true);
        }

        // 2桁目以降について、順列を閉路に分解、各閉路の情報を求める
        ArrayList<Integer> cycleId = new ArrayList<Integer>(d); // 各数がどの閉路に属するか
        ArrayList<Integer> cycleSize = new ArrayList<Integer>(); // 各閉路のサイズ
        ArrayList<Integer> index = new ArrayList<Integer>(d); // 各数が閉路内で何番目か
        for (int i = 0; i < d; i++) {
            cycleId.add(-1);
            index.add(-1);
        }
        for (int i = 0; i < d; i++) {
            if (cycleId.get(i) != -1)
                continue;
            int p = i;
            int cnt = 0;
            while (cycleId.get(p) == -1) {
                cycleId.set(p, cycleSize.size());
                index.set(p, cnt);
                p = b.get(p);
                cnt++;
            }
            cycleSize.add(cnt);
        }

        // 2桁目以降の各桁の数字だけをkと一致させるために必要なfの適用回数・周期を求める
        boolean ok = used0.get(kl.get(0));
        ArrayList<Long> ra = new ArrayList<Long>(); // 何回fを適用すれば一致するか
        ArrayList<Long> ma = new ArrayList<Long>(); // その周期
        ra.add((long) y0);
        ma.add((long) x0);

        for (int i = 1; i < ml.size(); i++) {
            if (cycleId.get(ml.get(i)) != cycleId.get(kl.get(i))) { // 何回fを適用してもその桁が一致しない場合
                ok = false;
                break;
            }
            int num = index.get(kl.get(i)) - index.get(ml.get(i));
            if (num < 0)
                num += cycleSize.get(cycleId.get(ml.get(i)));
            ra.add((long) num);
            ma.add((long) cycleSize.get(cycleId.get(ml.get(i))));
        }

        if (!ok) {
            return -1;
        }

        // CRTで解を求める
        long[] _ra = new long[ra.size()]; // arrayに変換
        long[] _ma = new long[ma.size()];
        for (int i = 0; i < ra.size(); i++) {
            _ra[i] = ra.get(i);
            _ma[i] = ma.get(i);
        }
        long[] ans = crt(_ra, _ma);

        if (ans[0] == 0 && ans[1] == 0) {
            return -1;
        } else {
            return ans[0];
        }
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        while (true) {
            // 入力ここから
            int d = scanner.nextInt();
            if (d == -1)
                break;
            ArrayList<Integer> a = new ArrayList<Integer>(d);
            ArrayList<Integer> b = new ArrayList<Integer>(d);
            a.add(0);
            for (int i = 1; i < d; i++) {
                a.add(scanner.nextInt());
            }
            for (int i = 0; i < d; i++) {
                b.add(scanner.nextInt());
            }
            BigInteger m = scanner.nextBigInteger();
            BigInteger k = scanner.nextBigInteger();
            // 入力ここまで

            long ans = solve(d, a, b, m, k);
            if (ans == -1) {
                System.out.println("NO");
            } else {
                System.out.println(ans);
            }
        }

        scanner.close();
    }
}

```