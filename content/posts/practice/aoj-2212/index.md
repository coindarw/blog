+++
title = "AOJ 2212 Stolen Jewel"
date = 2025-03-04T13:52:58+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2212

https://vjudge.net/problem/Aizu-2212
<!--more-->
## 問題概要
- $N\times M$のグリッドの迷路が与えられ、スタートからゴールまで移動する必要がある
- $P$種類の禁止パターンが存在し、それぞれ10文字以内のL,R,U,Dからなる文字列である
- 移動経路をL,R,U,Dで表した文字列の部分文字列として禁止パターンが入っていてはならない場合、ゴールまでたどり着けるか？またたどり着ける場合の移動回数の最小値はいくつか
### 制約
- $1\leq N,M\leq 50$
- $0\leq P\leq 10$
- 禁止パターンの文字列は1文字以上10文字以内

## 解法メモ
- もう少し制約が小さければ過去10回の移動を記録してDPなどすればいいが、このままではできない
- 実際にはすべての移動情報が必要なわけではない。今各禁止パターンにどれくらいマッチしているかが分かればいい。

- 禁止パターンでTrieを作り、禁止パターンに最長どこまでマッチしているかを持てばいい
- 最長禁止パターンから外れた場合は、末尾の最大10文字を愚直に調べればよい。
- 禁止パターンがもっと長い場合はAho-Corasickなどを使わなければならないが、今回は10文字以下なので適当にやっても間に合う

- (位置, Trieのノード)のペアを頂点とする有向グラフを作り、その上をBFSして最短距離を出力すればよい
## 実装例
Trieのインターフェースが固まっていない。
Aho-Corasickの劣化版のようなことをしているので素直にそちらを貼ったほうが楽かもしれない
```cpp
#include <bits/stdc++.h>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
constexpr int inf = 2000'000'000;
#define all(v) begin(v), end(v)
using namespace std;

template <int char_size, int base>
class Trie {
   public:
    struct Node {
        int next[char_size];
        vector<int> accept;  // 末端がこの頂点になる単語のword_idx
        int c;               // str[i] - base
        int common;          // 頂点を共有する単語数
        int parent = -1;     // 親ノード
        Node(int c) : c(c), common(0), parent(-1) { rep(i, char_size) next[i] = -1; }
    };
    vector<Node> nodes;

    Trie() { nodes.push_back(Node(0)); }
    void insert(const string &word, int word_idx) {  // word_idx番目の単語を追加
        int node_id = 0;
        for (int i = 0; i < word.size(); ++i) {
            int c = word[i] - base;
            if (nodes[node_id].next[c] == -1) {
                nodes[node_id].next[c] = int(nodes.size());
                nodes.push_back(Node(c));
                nodes.back().parent = node_id;
            }
            ++nodes[node_id].common;
            node_id = nodes[node_id].next[c];
        }
        ++nodes[node_id].common;
        nodes[node_id].accept.push_back(word_idx);
    }
    void insert(const string &word) { insert(word, nodes[0].common); }
    int search_id(const string &word) {  // プレフィックスとしてwordが存在する場合はノード番号を，存在しない場合は-1を返す
        int node_id = 0;
        for (int i = 0; i < word.size(); ++i) {
            char c_ = word[i];
            int c = c_ - base;
            int next_id = nodes[node_id].next[c];
            if (next_id == -1) return -1;
            node_id = next_id;
        }
        return node_id;
    }
    bool search(const string &word, bool prefix = false) {  // 単語wordが存在するかどうか, prefix=trueならwordがtrieに含まれるか
        int node_id = search_id(word);
        if (node_id == -1) return false;
        return prefix || (nodes[node_id].accept.size() > 0);
    }
    bool starts_with(const string &prefix) { return search(prefix, true); }
    int word_count() const { return nodes[0].common; }
    int node_size() const { return nodes.size(); }
    string operator[](int k) {  // k番目のノードの文字列を返す
        string ret;
        while (k != 0) {
            ret += char(nodes[k].c + base);
            k = nodes[k].parent;
        }
        reverse(ret.begin(), ret.end());
        return ret;
    }
};

char DIR[] = "LRUD";
const int dx[] = {-1, 1, 0, 0};
const int dy[] = {0, 0, -1, 1};
int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    while (true) {
        int n, m;
        cin >> n >> m;
        if (n == 0 && m == 0) break;
        vector<string> v(n);
        rep(i, n) cin >> v[i];
        int si, sj, gi, gj;
        rep(i, n) rep(j, m) {  // スタートとゴールの位置を探しておく
            if (v[i][j] == 'S') si = i, sj = j;
            if (v[i][j] == 'G') gi = i, gj = j;
        }

        int p;
        cin >> p;
        vector<string> banned(p);
        rep(i, p) cin >> banned[i];

        Trie<26, 'A'> trie;  // 禁止文字列のtrieを作る
        rep(i, p) trie.insert(banned[i]);
        vector<int> banned_node(p);
        rep(i, p) banned_node[i] = trie.search_id(banned[i]);

        // trieの各ノードから、各方向に進めるノードを探しておく
        // Aho-Corasickとほぼ同じことを愚直にやっているだけなのでライブラリがあるならそちらを使った方が良い
        int k = trie.node_size();
        vector<vector<int>> to(k, vector<int>(26, -1));
        rep(i, k) {
            string s = "";
            int cur = i;
            while (cur != 0) {
                s += char(trie.nodes[cur].c + 'A');
                cur = trie.nodes[cur].parent;
            }
            reverse(all(s));
            rep(ci, 4) {
                char c = DIR[ci];
                string t = s + c;
                rep(j, t.size() + 1) {
                    string u = t.substr(t.size() - j);  // tの末尾j文字
                    int res = trie.search_id(u);
                    if (res != -1) {
                        to[i][c - 'A'] = res;
                        if (trie.nodes[res].accept.size() > 0) {  // 移動先に禁止文字列が含まれる場合は進めない
                            to[i][c - 'A'] = -1;
                            break;
                        }
                    }
                }
            }
        }

        // BFSで最短距離を求める
        auto idx = [&](int i, int j, int node) { return i * m * k + j * k + node; };
        auto rev = [&](int idx) {
            int node = idx % k;
            idx /= k;
            int j = idx % m;
            idx /= m;
            int i = idx;
            return make_tuple(i, j, node);
        };
        vector<int> dist(n * m * k, inf);
        dist[idx(si, sj, 0)] = 0;
        queue<int> que;
        que.push(idx(si, sj, 0));
        while (!que.empty()) {
            auto [i, j, node] = rev(que.front());
            que.pop();
            rep(ci, 4) {
                char c = DIR[ci];
                int ni = i + dy[ci], nj = j + dx[ci];
                if (ni < 0 || ni >= n || nj < 0 || nj >= m) continue;
                int nNode = to[node][c - 'A'];
                int nIdx = idx(ni, nj, nNode);
                if (nNode == -1) continue;
                if (v[ni][nj] == '#') continue;
                if (dist[nIdx] > dist[idx(i, j, node)] + 1) {
                    dist[nIdx] = dist[idx(i, j, node)] + 1;
                    que.push(nIdx);
                }
            }
        }

        int ans = inf;
        rep(node, k) ans = min(ans, dist[idx(gi, gj, node)]);
        if (ans == inf) cout << -1 << "\n";
        else cout << ans << "\n";
    }
}
```