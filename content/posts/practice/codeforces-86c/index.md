+++
title = "Codeforces 86C Genetic Engineering"
date = 2025-03-04T13:53:07+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://codeforces.com/problemset/problem/86/C

https://vjudge.net/problem/CodeForces-86C
<!--more-->
## 問題概要
- 10文字以下、"A","T","C","G"4文字からなる$M$個の文字列$s_i$が与えられる
- $N$文字の"A","T","C","G"からなる文字列で、以下の条件を満たすものの個数を数えよ
	- 各位置$i$に対して$l\leq i\leq r$が存在し、文字列の$l$から$r$文字目が$s_i$いずれかに一致する

- 分かりづらいので例示すると、$s_i$として"GC", "AT", "TC"の3つが与えられたときに、"ATCGCGC"のように、重なりを許して$s_i$を並べることで作れる長さ$N$の文字列を数え上げる

### 制約
- $1\leq N\leq 1000$
- $1\leq M\leq 10$
- $|s_i|\leq 10$
## 解法メモ
- Aho-Corasickで解くことができる

- DPテーブルの定義は以下のような感じ
	- `dp[i][j][k] = i文字目まで決めて、trieノードの中でサフィックスと一致する最長のものがjで、末尾k文字が被覆されていないものの数`

- DPで1文字ずつ末尾に追加していく
- 遷移先のノードの末尾が与えられた単語のどれかに一致し、うまく被覆できるならkの値を0にすればよい
## 実装例
```cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <queue>
#include <vector>
#define rep(i, n) for (int i = 0, i##_len = (n); i < i##_len; ++i)
constexpr int M09 = 1'000'000'000 + 9;
using namespace std;

template <int char_size, int base>
class AhoCorasick {
   public:
    struct Node {
        int next[char_size];
        vector<int> accept;  // 末端がこの頂点になる単語のword_idx
        int c;               // str[i] - base
        int common;          // 頂点を共有する単語数
        int parent;          // 親ノード
        int depth;
        Node(int _c) : c(_c), common(0), parent(-1), depth(0) {
            for (int i = 0; i < char_size; ++i) next[i] = -1;
        }
    };

   private:
    vector<Node> _nodes;
    vector<int> _suffix;
    vector<int> _accepted_prefix, _accepted_suffix;
    bool built = false;
    void insert(const string &word, int word_idx) {  // word_idx番目の単語を追加
        assert(!word.empty());
        int node_id = 0;
        for (int i = 0; i < (int)word.size(); ++i) {
            int c = word[i] - base;
            if (_nodes[node_id].next[c] == -1) {
                _nodes[node_id].next[c] = int(_nodes.size());
                _nodes.push_back(Node(c));
                _nodes.back().parent = node_id;
                _nodes.back().depth = _nodes[node_id].depth + 1;
            }
            ++_nodes[node_id].common;
            node_id = _nodes[node_id].next[c];
        }
        ++_nodes[node_id].common;
        _nodes[node_id].accept.push_back(word_idx);
    }

   public:
    AhoCorasick() { _nodes.push_back(Node(0)); }
    void insert(const string &word) { insert(word, _nodes[0].common); }
    int search_id(const string &word) {  // プレフィックスとしてwordが存在する場合はノード番号を，存在しない場合は-1を返す
        int node_id = 0;
        for (int i = 0; i < (int)word.size(); ++i) {
            char c_ = word[i];
            int c = c_ - base;
            int next_id = _nodes[node_id].next[c];
            if (next_id == -1) return -1;
            node_id = next_id;
        }
        return node_id;
    }
    bool search(const string &word,
                bool prefix = false) {  // 単語wordが存在するかどうか,
                                        // prefix=trueならwordがtrieに含まれるか
        int node_id = search_id(word);
        if (node_id == -1) return false;
        return prefix || (_nodes[node_id].accept.size() > 0);
    }
    bool starts_with(const string &prefix) { return search(prefix, true); }
    int word_count() const { return _nodes[0].common; }
    int node_size() const { return _nodes.size(); }
    string operator[](int k) {  // k番目のノードの文字列を返す
        string ret;
        while (k != 0) {
            ret += char(_nodes[k].c + base);
            k = _nodes[k].parent;
        }
        reverse(ret.begin(), ret.end());
        return ret;
    }
    void build() {
        _suffix.assign(node_size(), 0);
        _accepted_prefix.assign(node_size(), -1);
        _accepted_suffix.assign(node_size(), -1);
        queue<int> que;
        for (int i = 0; i < char_size; ++i) {
            int next_id = _nodes[0].next[i];
            if (next_id == -1) {
                _nodes[0].next[i] = 0;
                continue;
            }
            que.push(next_id);
            _suffix[next_id] = 0;
        }
        while (!que.empty()) {
            int node_id = que.front();
            que.pop();
            int c = _nodes[node_id].c;
            int parent = _nodes[node_id].parent;
            bool is_accept = !_nodes[node_id].accept.empty();
            if (parent != 0) {  // len != 1
                _suffix[node_id] = _nodes[_suffix[parent]].next[c];
                _accepted_suffix[node_id] = _nodes[_suffix[node_id]].accept.empty() ? _accepted_suffix[_suffix[node_id]] : _suffix[node_id];
            }
            for (int i = 0; i < char_size; ++i) {
                int next_id = _nodes[node_id].next[i];
                if (next_id == -1) {
                    _nodes[node_id].next[i] = _nodes[_suffix[node_id]].next[i];
                } else {
                    que.push(next_id);
                    if (is_accept) _accepted_prefix[next_id] = node_id;
                    else _accepted_prefix[next_id] = _accepted_prefix[node_id];
                }
            }
        }
        built = true;
    }
    const vector<Node> &nodes() { return _nodes; }
    const vector<int> &suffix() {
        assert(built);
        return _suffix;
    }
    const vector<int> &accepted_suffix() {
        assert(built);
        return _accepted_suffix;
    }
    const vector<int> &accepted_prefix() {
        assert(built);
        return _accepted_prefix;
    }
};

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int n, m;
    cin >> n >> m;
    AhoCorasick<4, 'A'> ac;
    rep(i, m) {
        string s;
        cin >> s;
        rep(i, s.size()) {  // AGCT -> ABCD
            if (s[i] == 'G') s[i] = 'B';
            if (s[i] == 'T') s[i] = 'D';
        }
        ac.insert(s);
    }
    ac.build();

    int ns = ac.node_size();
    // dp[i][j][k] = i文字目まで決めて、
    //               trieノードとサフィックスが一致する最長のものがjで、
    //               末尾k文字が被覆されていないものの数
    vector<vector<vector<int>>> dp(n + 1, vector<vector<int>>(ns, vector<int>(11)));
    dp[0][0][0] = 1;

    rep(i, n) {
        rep(j, ns) {
            rep(k, 11) {
                if (dp[i][j][k] == 0) continue;
                rep(c, 4) {
                    int nxt = ac.nodes()[j].next[c];
                    int nk = k + 1;
                    // 今いるノードが単語の末尾になった
                    if (!ac.nodes()[nxt].accept.empty()) {
                        if (ac.nodes()[nxt].depth >= nk) {
                            nk = 0;
                        }
                    }
                    // 今いるノードの深さより短いサフィックスが単語の末尾になった
                    if (ac.accepted_suffix()[nxt] != -1) {
                        if (ac.nodes()[ac.accepted_suffix()[nxt]].depth >= nk) {
                            nk = 0;
                        }
                    }
                    if (nk > 10) continue;
                    dp[i + 1][nxt][nk] += dp[i][j][k];
                    dp[i + 1][nxt][nk] %= M09;
                }
            }
        }
    }
    int ans = 0;

    rep(j, ns) {
        ans += dp[n][j][0];
        ans %= M09;
    }
    cout << ans << "\n";
}
```