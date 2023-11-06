+++
title = '競プロ関連、C++のちょっとしたこと'
date = 2023-11-05T01:39:58+09:00
tags = ['C++', '競技プログラミング']
+++

競技プログラミングに関係したC++関連のTips的なものまとめ。あくまで競プロ用なので、業務ではもっと別の方法を取ったほうがよかったりすると思う。

環境としてはWindows, WSL or MinGWでGCCを使うことを想定しているが、所々MinGWでは動かないところもあるので注意。検証していないがLinuxならたぶん大丈夫だと思う。clangはほぼ使ったことがないので不明。

<!--more-->

## 未定義動作の検出

配列範囲外参照、0除算など、未定義動作をコードに仕込んでしまうと、手元環境では正しい答えを出すのに提出するとWA/REが出るといったことが起きる。

このような場合の簡単な対処法をこの項と次項「デバッガの使用によるスタックトレースの表示」に書いていく

以下のようなコンパイルオプションを指定することでかなり多くの未定義動作を検出することができる。

（WindowsでMinGWを使っている場合は `fsanitize` が使えないので注意。WSLを使うといい）

```bash
g++ -fsanitize=undefined,address -D_GLIBCXX_DEBUG 
```

かなり多くのオプションがあるが、とりあえず3つだけ書く。

これ以外にもメモリリークを検知する `LeakSanitizer` とかいろいろあるので調べると面白そう

https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html#index-fsanitize_003daddress

### fsanitize=undefined

WindowsのMinGWでは動かないので注意。WSLなら大丈夫。[clangでもいける](https://stackoverflow.com/questions/67619314/cannot-use-fsanitize-address-in-mingw-compiler)っぽい？

未定義動作サニタイザ

- ゼロ除算
- オーバーフロー
- 範囲外のシフト操作

などを検出してくれる

### fsanitize=address

これもMinGWでは動かない

不正なメモリ操作を検知してくれる

生配列・vectorへの `[]` アクセスの範囲外参照やスタックオーバーフローなど

### D_GLIBCXX_DEBUG

これはMinGWでも使える。

これを付けておくと、_GLIBCXX_DEBUGマクロが定義されて標準ライブラリがデバッグ用のものに置き換わる。

具体例としては、 `vector` で `[]` アクセスする際に境界チェックが行われる、 `lower_bound` を実行した際に配列などの中身の順番がおかしくなっていない（[partitionedである](https://cpprefjp.github.io/reference/algorithm.html#sequence-is-partitioned)）かチェックするなど。

注意：なお、これを付けるとエラー検知のために計算量も変わることがあるので、提出コードに `#define _GLIBCXX_DEBUG` などとするとTLEしてしまったりする

余談：lower_boundの要件がソート済みであることではなく、区分化されている（sequence is partitioned）ことにC++11から変わっているの知らなかった

## デバッガ(gdb)の使用によるスタックトレースの表示

注意：以降の手順で、WSL1を使用している場合、正しくスタックトレースが表示できない場合がある。（ `#0  0x0000000000000000 in ?? ()` などと出てくる）    `wsl -l -v` を実行し、VERSIONの欄が1となっている場合はWSL2にアップデートしておくこと。

様々な機能があるのだが、とりあえず競プロ中に一番欲しくなるであろう機能として、スタックトレースを表示する方法を書く。

何行目のどの関数の中でエラーが起きたのか確認することができる

gdbを使うのではなく、環境変数 `UBSAN_OPTIONS` に `print_stacktrace=1` などと指定してもよさそう

また、この項ではコマンドラインでの操作を前提としているが、学生であればGithub Educationに登録することで統合開発環境のClionが無料で使える。GUIでデバッグできるのでこれを使ってもいいと思う

### 手順

以下の提出のデバッグを行いたいとする。　TODO リンク追加

1. コンパイルする際、オプションとして `-g` を付けて実行ファイルにデバッグ用の情報が入るようにする。
    1. この際、前項のデバッグ用のオプションも併用するとよい。
    2. また、 `-O0` で最適化を切っておく
    3. 長くなるのでエイリアスしておくとよい

```bash
g++ -fsanitize=undefined,address -D_GLIBCXX_DEBUG main.cpp -g -O0
```

2. gdbを実行（この時点では `a.out` は実行されない）

```bash
gdb a.out
```

3. gdbの操作画面になるので、 `run` でプログラムを実行する
    1. この時自分で入力を打ち込んだり、コピペしたりしてもいいのだが、入力途中でプログラムが終了するとそれ以降の入力が入ってしまって不便。入力ファイルを作って `< in.txt` のようにリダイレクトで渡すのが良いと思う

```bash
run < in.txt
```

4. 途中で落ちた場合、 `bt` でトレースを表示する。（Backtraceの略らしい）
    1. なお、深い再帰の中でエラーが起きた場合などはとんでもない長さになる。 `bt 10` のように最大でいくつ表示するか指定できる
    2. `bt 10` だと奥から10個、 `bt -10` だと手前（例えばmainは普通手前側）から10個表示される

```bash
bt
```

5. 終了する際は `q` を入力する。 `Quit anyway? (y or n)` と聞かれるのでyを入力すると終了できる

```bash
q
```

## デバッグ出力

まず、標準エラー出力をデバッグ用出力として使用することがよく行われる

C++であれば以下のようにすることで、答えには影響を与えないで変数aの中身を出力することができる。

```cpp
cerr << a << endl;
```

しかし、以下のような欠点もあるため、少し工夫したデバッグ用マクロを作ることも多い

- 出力自体に時間がかかるため、ループ内で出力などしてしまうとTLEになってしまうことがある
- ジャッジによっては標準エラー出力で余計なアウトプットをしてしまうと正解扱いにならない

### デバッグ用マクロ

C++では `#ifdef` のようなプリプロセッサ命令を使うことで、環境ごとにコードを変えることができる。

多くのジャッジ（AtCoder, Codeforces, yukicoderなど）ではC/C++のコンパイルコマンドに `-DONLINE_JUDGE` というオプションがついており、ONLINE_JUDGEというマクロが存在する扱いになっている。

もしくはジャッジ側ではなく、ローカル環境の方でコンパイルする際に `-DLOCAL` オプションなどを付けて特別扱いする場合もある。別PCを使用する際にエイリアス設定をする必要があるなど手間は多少あるが、どのオンラインジャッジでも使えることが利点。

例えば以下のようなコードを書けば、手元の環境では `debug(a)` と書くとaの値を出力してくれるが、提出した場合は実行に一切影響しない（余計な実行時間もかからない）デバッグ用マクロを定義することができる。

実際には、複数変数の出力などに対応させるため、もう少し複雑なコードにしている人が多いと思う

```cpp
#ifdef ONLINE_JUDGE
#define debug(x)
#else
#define debug(x) cerr << x << endl;
#endif
```

~~私のデバッグ用マクロは特定のコンテナを入れると壊れたりするのでここには載せません。~~ 

「競プロ　デバッグ　マクロ」とかで検索すると強い人のテンプレがいろいろ出てくる。

## bits/stdc++.hのプリコンパイル

主要な標準ライブラリをまとめてincludeしてくれるbits/stdc++.hは便利だが、コンパイル時間が伸びてしまう。

bits/stdc++.hだけ事前にコンパイルしておき、コンパイル時にはこれを参照するようにしておくことでかなりコンパイル時間が短縮される。

なお、コンパイルオプションごとに別にプリコンパイルしておくことが必要。

`#pragma` 命令の扱いがよくわかっていない．

 `#include` の後に `#pragma` があればプリコンパイルヘッダが使われるが、逆だと使われないなど条件がよくわからない

1回のコンパイルにつき、プリコンパイルヘッダは1つしか指定できないので、 `<bits/stdc++.h>` と `<atcoder/all>` の両方をプリコンパイルしておくといったことはできない

> • Only one precompiled header can be used in a particular compilation.
> 

私の場合はそれらをまとめた `<mylibs/all.h>` をプリコンパイルしておき、 `ONLINE_JUDGE` マクロの有無でどちらをインクルードするか分岐している。WSLで1.3秒くらいでコンパイルできる。

プリコンパイルヘッダーとしてコンパイル時に認識される条件などは割と複雑。ドキュメントの該当ページは短いので通して読んでおくのがよさそう

https://gcc.gnu.org/onlinedocs/gcc/Precompiled-Headers.html

`.h.gch` ファイルの名前は基本何でもいいが，ディレクトリ名や拡張子を間違えると認識してくれないので注意

### 手順

Linux/WSLであればこんな感じ

（fatal error … Permission deniedなどと言われたら適宜 `sudo` を付けて実行）

```bash
# bits/stdc++.hの場所を調べる
echo -e "#include<bits/stdc++.h>\n int main(){}" | g++ -xc++ - -M | grep "bits/stdc++.h"

cd /usr/include/x86_64-linux-gnu/c++/11/bits # 上のコマンドで得られた場所に適宜書き換え
mkdir stdc++.h.gch
cd stdc++.h.gch

# 使うコンパイルコマンドごとにコンパイル
g++ stdc++.h -o stdc++.h.gch/stdc++.h.gch
g++ stdc++.h -o stdc++.h.gch/stdc++_o2.h.gch -O2
g++ stdc++.h -o stdc++.h.gch/stdc++_debug.h.gch -fsanitize=undefined,address -D_GLIBCXX_DEBUG -g -O0
```

Windowsであればこんな感じ

```powershell
# bits/stdc++.hの場所を調べる
echo  "#include<bits/stdc++.h>`nint main(){}" > bb.cpp; g++ bb.cpp -M |  Select-String -SimpleMatch "bits/stdc++"

# 適宜得られたパスに書き換え、bitsディレクトリまで行く
cd C:/mingwのパス/lib/gcc/x86_64-w64-mingw32/13.1.0/include/c++/x86_64-w64-mingw32
mkdir stdc++.h.gch
cd stdc++.h.gch

# 使うコンパイルコマンドごとにコンパイル
g++ stdc++.h -o stdc++.h.gch/stdc++.h.gch
g++ stdc++.h -o stdc++.h.gch/stdc++_o2.h.gch -O2
g++ stdc++.h -o stdc++.h.gch/stdc++_debug.h.gch -D_GLIBCXX_DEBUG -g -O0
```

### 確認方法

（実際のところ、目に見えてコンパイル時間は縮むので基本的には設定できたことは感覚でわかるのだが、ファイル名など間違えているとうまくいかないので）

適当なファイルを作って、そこでプリコンパイルしたヘッダをインクルードする

```cpp
#include <bits/stdc++.h>
int main () {
}
```

`-H` オプションを使うとインクルードしたヘッダを確認できる．例えば次のような出力が出て、現在の設定に合致するオプションでコンパイルしたファイルが勝手に選ばれることが確認できる。

```bash
$ g++ main.cpp -H
x /usr/include/x86_64-linux-gnu/c++/11/bits/stdc++.h.gch/stdc++_debug.h.gch
x /usr/include/x86_64-linux-gnu/c++/11/bits/stdc++.h.gch/stdc++_o2_debug.h.gch
x /usr/include/x86_64-linux-gnu/c++/11/bits/stdc++.h.gch/stdc++_o3.h.gch
! /usr/include/x86_64-linux-gnu/c++/11/bits/stdc++.h.gch/stdc++.h.gch
 main.cpp
```

上手く設定できていないと、このように大量のファイルを読みに行ってしまう

```bash
$ g++ main.cpp -H
. /usr/include/x86_64-linux-gnu/c++/11/bits/stdc++.h
.. /usr/include/c++/11/cassert
... /usr/include/x86_64-linux-gnu/c++/11/bits/c++config.h
.... /usr/include/x86_64-linux-gnu/c++/11/bits/os_defines.h
..... /usr/include/features.h
...... /usr/include/features-time64.h
....... /usr/include/x86_64-linux-gnu/bits/wordsize.h
....... /usr/include/x86_64-linux-gnu/bits/timesize.h
........ /usr/include/x86_64-linux-gnu/bits/wordsize.h
...... /usr/include/x86_64-linux-gnu/sys/cdefs.h
....... /usr/include/x86_64-linux-gnu/bits/wordsize.h
....... /usr/include/x86_64-linux-gnu/bits/long-double.h
...... /usr/include/x86_64-linux-gnu/gnu/stubs.h
....... /usr/include/x86_64-linux-gnu/gnu/stubs-64.h
.... /usr/include/x86_64-linux-gnu/c++/11/bits/cpu_defines.h
.... /usr/include/c++/11/pstl/pstl_config.h
... /usr/include/assert.h
.. /usr/include/c++/11/cctype

...以下大量に続く
```

## エイリアス

デバッグ用のオプションを盛ったコンパイルコマンドなどはかなり長くなるので、エイリアス機能を使って短いコマンドを作っておくと便利

WindowsのPowerShellであれば `$PROFILE` がプロファイルのパスになっているので、

```powershell
code $PROFILE # VSCodeでプロファイルを開く
```

などとして以下のようにエイリアスを定義することができる

この例では `g++debug` というエイリアスを定義している

```powershell
function g++debug {
    & 'g++.exe' '-std=c++20' '-D_GLIBCXX_DEBUG' '-g' '-O0' @args
}
```

Linux/WSLであれば`.bashrc` などに以下のように記述することでエイリアスを定義できる。

```bash
alias g++debug='g++ -fsanitize=undefined,address -D_GLIBCXX_DEBUG -g -O0'
```

## その他便利なオプションなど

### コンパイルオプション・pragmaなど

- `-Wfatal-errors`
    - 最初の1つ目のコンパイルエラーが発生した時点でコンパイルを止める
- `#pragma GCC diagnostic error "警告"`
    - 特定の警告をエラーにすることができる。例えば`#pragma GCC diagnostic error "-Wshadow"` とテンプレートの上の方に入れておけば、変数のシャドウイングをエラーにすることができ、ありがちな「入力にkがあるのに添字でもkを使ってしまった」のようなことを防止できる。
    - コンパイルオプション `-Werror=shadow` でもいいが、ソースコードに入れておくとCE扱いになるので安全
    
    ```cpp
    int n, k; // 入力にkがあるのに
    cin >> n >> k;
    rep(i, n) rep(j, n) rep(k, n) { // 添字としてkを使ってしまっている
        /* 何かの処理 */
    }
    ```
    
- `export UBSAN_OPTIONS="print_stacktrace=1"`
    - 環境変数で指定しておくとサニタイザを指定してコンパイルしたプログラムが落ちた時にスタックトレースを表示してくれる
    - `-g` オプションも併用するとスタックトレースの行数も出る
    - 「未定義動作の検出」の項にあるようにWindowsのMinGWでは使えない
    - 以下参照
        - https://taotao54321.hatenablog.com/entry/2021/02/12/075342

### 便利な文法・関数

- 構造化束縛・ `std::tie`
- `std::is_sorted`
- `std::static_assert`

### 作っておくと便利な関数など

- [多次元vectorを作るマクロ](https://www.google.com/search?q=%E5%A4%9A%E6%AC%A1%E5%85%83vector+%E3%83%9E%E3%82%AF%E3%83%AD+C%2B%2B+%E7%AB%B6%E3%83%97%E3%83%AD)
- YesNo関数：`string YesNo(bool cond) { return cond ? "Yes" : "No"; }`
- 配列全部に1足したりできる `auto add(auto vec, ll x = 1) { for (auto& e : vec) e += x; return vec; }`
- ACLのmodintの出力のオーバーライド `template <int M>inline ostream& operator<<(ostream& os, const static_modint<M>& m) { return os << m.val(); }`

### さらに些末なこと

- TODOマクロ。配列サイズをテスト用に小さくしたときなどでも、 `todo("Nを戻す");`などとしておくとSubmitしてもCEにしてくれる。

```cpp
#ifdef ONLINE_JUDGE
#define todo(err) static_assert(false, err)
#else
#define todo(err)
#endif
```

- 今はまだ使えないが、恐らく次の言語アプデで入るGCC14では `std::println` が使える。
    - https://cpprefjp.github.io/reference/print/println.html

## 参考

https://gcc.gnu.org/onlinedocs/gcc-6.3.0/gcc/Warning-Options.html#Warning-Options

https://aki33524.hatenablog.com/entry/2017/01/16/212526

https://taotao54321.hatenablog.com/entry/2021/02/12/075342

## 上で言及していないが便利なリンク集

https://qiita.com/TumoiYorozu/items/0ca14c65559e693f9e89

https://www.youtube.com/watch?v=uhnASau7fB4&list=PLAYMgc8c_QezzZAEcnhI_Awo1QHWxE6FD&index=3

- オプション・環境
    - AtC https://atcoder.jp/contests/abc314/rules （コンテストページ一番下の「ルール」）
    - CF https://codeforces.com/blog/entry/121114
    - yuki https://yukicoder.me/help/environments
    - AOJ https://onlinejudge.u-aizu.ac.jp/system_info
    - POJ http://poj.org/page?id=1000
    - HackerRank https://candidatesupport.hackerrank.com/hc/en-us/articles/4402913877523-Execution-Environment-and-Samples
    - paiza https://paiza.jp/guide/language
