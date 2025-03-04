+++
title = "UVA 10181 15-Puzzle Problem"
date = 2025-03-04T13:54:27+09:00
tags = ['競技プログラミング', '蟻本練習問題']
+++

https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=1122

https://vjudge.net/problem/UVA-10181
<!--more-->
## 問題概要
- 15パズルを解いて、操作列を出力せよ（最小手数で解く必要はない）
- ただし、45ステップ以内で解けないなら「This puzzle is not solvable.」と出力せよ
## 解法メモ
- AOJのCoursesでも出てくる問題

- A\*やIDA\* などを実装すればよさそう

- なお、以下のページに書いてあるが、奇置換だと解けないのでパリティを求めれば早期リターンが可能
	- [8パズル，15パズルの不可能な配置と判定法 | 高校数学の美しい物語 (manabitimes.jp)](https://manabitimes.jp/math/979)
## 解法
A\*、IDA\*両方実装してみた。IDA\*の方が数倍速い

IDA\*
```cpp
#include <algorithm>
#include <array>
#include <cassert>
#include <cmath>
#include <cstdint>
#include <cstdio>
#include <iostream>
#include <map>
#include <vector>
#define rep(i, n) for (int i = 0; i < int(n); ++i)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
using namespace std;
#define all(v) v.begin(), v.end()

// uint64を配列として使うクラス
template <uint64_t MAX_VALUE, size_t LENGTH>
struct Array {
    static_assert(MAX_VALUE > 0, "MAX_VALUE must be greater than 0");

   private:
    constexpr static size_t bit_length(uint64_t value) {
        return (value <= 1) ? 1 : 1 + bit_length(value >> 1);
    }
    constexpr static size_t max_bit_length(uint64_t max_value) { return bit_length(max_value); }

   public:
    // 1要素が占めるビット数
    constexpr static size_t BIT_LENGTH = max_bit_length(MAX_VALUE);
    static_assert(BIT_LENGTH < 32, "BIT_LENGTH must be less than 32");
    // 64bit整数の中で何要素分あるか
    constexpr static size_t BLOCK_LENGTH = 64 / BIT_LENGTH;
    // 用意する配列の長さ
    constexpr static size_t ARRAY_LENGTH = (LENGTH + BLOCK_LENGTH - 1) / BLOCK_LENGTH;
    std::array<uint64_t, ARRAY_LENGTH> data = {};

    Array() {}
    Array(const std::array<uint64_t, ARRAY_LENGTH>& data) : data(data) {}
    Array(const Array& other) : data(other.data) {}
    Array(Array&& other) : data(std::move(other.data)) {}
    Array& operator=(const Array& other) {
        data = other.data;
        return *this;
    }
    Array& operator=(Array&& other) {
        data = std::move(other.data);
        return *this;
    }

    // operator[]でアクセス（取り出すだけ）
    uint32_t operator[](size_t index) const {
        return (data[index / BLOCK_LENGTH] >> (index % BLOCK_LENGTH * BIT_LENGTH)) &
               ((uint32_t(1) << BIT_LENGTH) - 1);
    }
    struct Proxy {
        Array& bitArray;
        size_t index;
        operator uint32_t() const {
            return (bitArray.data[index / BLOCK_LENGTH] >> (index % BLOCK_LENGTH * BIT_LENGTH)) &
                   ((uint32_t(1) << BIT_LENGTH) - 1);
        }
        Proxy& operator=(uint32_t value) {
            assert(value <= MAX_VALUE);
            size_t block = index / BLOCK_LENGTH;
            size_t offset = index % BLOCK_LENGTH * BIT_LENGTH;
            bitArray.data[block] &= ~(((uint64_t(1) << BIT_LENGTH) - 1) << offset);
            bitArray.data[block] |= (uint64_t(value) << offset);
            return *this;
        }
        Proxy& operator=(const Proxy& other) { return *this = uint32_t(other); }
        Proxy& operator=(Proxy&& other) { return *this = uint32_t(other); }
        Proxy(const Proxy& other) : bitArray(other.bitArray), index(other.index) {}
        Proxy(Proxy&& other) : bitArray(other.bitArray), index(other.index) {}
        Proxy(Array& bitArray, size_t index) : bitArray(bitArray), index(index) {}
    };
    // operator[]でアクセス（読み取りおよび書き込み）
    Proxy operator[](size_t index) { return Proxy{*this, index}; }
    void set(size_t index, uint32_t value) {
        assert(value <= MAX_VALUE);
        size_t block = index / BLOCK_LENGTH;
        size_t offset = index % BLOCK_LENGTH * BIT_LENGTH;
        data[block] &= ~(((uint64_t(1) << BIT_LENGTH) - 1) << offset);
        data[block] |= (uint64_t(value) << offset);
    }
    bool operator<(const Array& other) const {
        for (size_t i = 0; i < ARRAY_LENGTH; ++i)
            if (data[i] != other.data[i]) return data[i] < other.data[i];
        return false;
    }
    bool operator==(const Array& other) const {
        for (size_t i = 0; i < ARRAY_LENGTH; ++i)
            if (data[i] != other.data[i]) return false;
        return true;
    }
    bool operator!=(const Array& other) const { return !(*this == other); }
    friend std::ostream& operator<<(std::ostream& os, const Array& array) {
        for (size_t i = 0; i < LENGTH; ++i) {
            if (i) os << ' ';
            os << array[i];
        }
        return os;
    }
    friend std::istream& operator>>(std::istream& is, Proxy proxy) {
        uint32_t value;
        is >> value;
        proxy = value;
        return is;
    }
};

using State = Array<15, 16>;

void print(State state) {
    rep(i, 4) {
        rep(j, 4) cout << "123456789ABCDEF-"[state[i * 4 + j]] << " ";
        cout << endl;
    }
}

vector<State> next_states(const State& state) {
    vector<State> states;
    states.reserve(4);
    const int dx[] = {0, 1, 0, -1};
    const int dy[] = {1, 0, -1, 0};
    int hole = 0;
    rep(i, 16) {
        if (state[i] == 15) {
            hole = i;
            break;
        }
    }
    int y = hole / 4, x = hole % 4;
    rep(k, 4) {
        int ny = y + dy[k], nx = x + dx[k];
        if (ny < 0 || ny >= 4 || nx < 0 || nx >= 4) continue;
        State new_state = state;
        new_state[hole] = state[ny * 4 + nx];
        new_state[ny * 4 + nx] = 15;
        states.emplace_back(new_state);
    }
    return states;
}

short h_star(const State& state) {
    short h = 0;
    rep(i, 16) {
        if (state[i] == 15) continue;
        int y = i / 4, x = i % 4;
        int yy = (state[i]) / 4, xx = (state[i]) % 4;
        h += abs(y - yy) + abs(x - xx);
    }
    return h;
}

map<State, short> memo;
// DFS
template <typename State, typename Cost, vector<State> (*next_states)(const State& state),
          Cost (*h_star)(const State& state)>
vector<State> dfs(State state, State goal, Cost g, Cost limit, Cost& next_limit) {
    Cost f = g + h_star(state);
    if (f > limit) {
        next_limit = min(next_limit, f);
        return {};
    }
    if (state == goal) return {state};
    if (memo.count(state) && memo[state] <= g) return {};
    memo[state] = g;
    vector<State> states = next_states(state);
    for (const State& next : states) {
        vector<State> path =
            dfs<State, Cost, next_states, h_star>(next, goal, g + 1, limit, next_limit);
        if (!path.empty()) {
            path.emplace_back(state);
            return path;
        }
    }
    return {};
}

// IDA*
template <typename State, typename Cost, vector<State> (*next_states)(const State& state),
          Cost (*h_star)(const State& state)>
vector<State> ida_star(State start, State goal) {
    Cost limit = h_star(start);
    const Cost MAX_LIMIT = 45;
    while (limit < MAX_LIMIT) {
        memo.clear();
        Cost next_limit = MAX_LIMIT + 1;
        vector<State> path =
            dfs<State, Cost, next_states, h_star>(start, goal, 0, limit, next_limit);
        reverse(path.begin(), path.end());
        if (!path.empty()) return path;
        if (next_limit > MAX_LIMIT) return {};
        limit = next_limit;
    }
    return {};
}

int main() {
    int t;
    cin >> t;
    while (t--) {
        State start, goal;
        rep(i, 16) cin >> (start[i]);
        rep(i, 16) {
            if (start[i] == 0) start[i] = 15;
            else start[i] = start[i] - 1;
        }
        rep(i, 16) goal[i] = i;

        // 順列の偶奇と空白マスの位置の偶奇が一致していれば解ける
        int inv = 0;
        rep(i, 16) rep2(j, i + 1, 16) {
            if (start[i] > start[j]) inv++;
        }
        int vy = -1, vx = -1;
        rep(i, 16) if (start[i] == 15) vy = i / 4, vx = i % 4;

        if (inv % 2 != (abs(vy - 3) + abs(vx - 3)) % 2) {
            cout << "This puzzle is not solvable." << endl;
            continue;
        }

        if (start == goal) {
            cout << endl;
            continue;
        }

        vector<State> path = ida_star<State, short, next_states, h_star>(start, goal);
        if (path.empty()) {
            cout << "This puzzle is not solvable." << endl;
        } else {
            string ans;
            int prvY = -1, prvX = -1;
            rep(i, path.size()) {
                rep(y, 4) {
                    rep(x, 4) {
                        if (path[i][y * 4 + x] == 15) {
                            if (prvY != -1) {
                                if (prvY == y) {
                                    if (prvX < x) {
                                        ans += "R";
                                    } else {
                                        ans += "L";
                                    }
                                } else {
                                    if (prvY < y) {
                                        ans += "D";
                                    } else {
                                        ans += "U";
                                    }
                                }
                            }
                            prvY = y;
                            prvX = x;
                        }
                    }
                }
            }

            cout << ans << endl;
        }
    }
}
```


```cpp
#include <algorithm>
#include <array>
#include <cassert>
#include <cmath>
#include <cstdint>
#include <cstdio>
#include <iostream>
#include <map>
#include <queue>
#include <vector>
#define rep(i, n) for (int i = 0; i < int(n); ++i)
#define rep2(i, s, n) for (int i = (s); i < (int)(n); i++)
using namespace std;
#define all(v) v.begin(), v.end()

template <uint64_t MAX_VALUE, size_t LENGTH>
struct Array {
    static_assert(MAX_VALUE > 0, "MAX_VALUE must be greater than 0");

   private:
    constexpr static size_t bit_length(uint64_t value) {
        return (value <= 1) ? 1 : 1 + bit_length(value >> 1);
    }
    constexpr static size_t max_bit_length(uint64_t max_value) { return bit_length(max_value); }

   public:
    // 1要素が占めるビット数
    constexpr static size_t BIT_LENGTH = max_bit_length(MAX_VALUE);
    static_assert(BIT_LENGTH < 32, "BIT_LENGTH must be less than 32");
    // 64bit整数の中で何要素分あるか
    constexpr static size_t BLOCK_LENGTH = 64 / BIT_LENGTH;
    // 用意する配列の長さ
    constexpr static size_t ARRAY_LENGTH = (LENGTH + BLOCK_LENGTH - 1) / BLOCK_LENGTH;
    std::array<uint64_t, ARRAY_LENGTH> data = {};

    Array() {}
    Array(const std::array<uint64_t, ARRAY_LENGTH>& data) : data(data) {}
    Array(const Array& other) : data(other.data) {}
    Array(Array&& other) : data(std::move(other.data)) {}
    Array& operator=(const Array& other) {
        data = other.data;
        return *this;
    }
    Array& operator=(Array&& other) {
        data = std::move(other.data);
        return *this;
    }

    // operator[]でアクセス（取り出すだけ）
    uint32_t operator[](size_t index) const {
        return (data[index / BLOCK_LENGTH] >> (index % BLOCK_LENGTH * BIT_LENGTH)) &
               ((uint32_t(1) << BIT_LENGTH) - 1);
    }
    struct Proxy {
        Array& bitArray;
        size_t index;
        operator uint32_t() const {
            return (bitArray.data[index / BLOCK_LENGTH] >> (index % BLOCK_LENGTH * BIT_LENGTH)) &
                   ((uint32_t(1) << BIT_LENGTH) - 1);
        }
        Proxy& operator=(uint32_t value) {
            assert(value <= MAX_VALUE);
            size_t block = index / BLOCK_LENGTH;
            size_t offset = index % BLOCK_LENGTH * BIT_LENGTH;
            bitArray.data[block] &= ~(((uint64_t(1) << BIT_LENGTH) - 1) << offset);
            bitArray.data[block] |= (uint64_t(value) << offset);
            return *this;
        }
        Proxy& operator=(const Proxy& other) { return *this = uint32_t(other); }
        Proxy& operator=(Proxy&& other) { return *this = uint32_t(other); }
        Proxy(const Proxy& other) : bitArray(other.bitArray), index(other.index) {}
        Proxy(Proxy&& other) : bitArray(other.bitArray), index(other.index) {}
        Proxy(Array& bitArray, size_t index) : bitArray(bitArray), index(index) {}
    };
    // operator[]でアクセス（読み取りおよび書き込み）
    Proxy operator[](size_t index) { return Proxy{*this, index}; }
    void set(size_t index, uint32_t value) {
        assert(value <= MAX_VALUE);
        size_t block = index / BLOCK_LENGTH;
        size_t offset = index % BLOCK_LENGTH * BIT_LENGTH;
        data[block] &= ~(((uint64_t(1) << BIT_LENGTH) - 1) << offset);
        data[block] |= (uint64_t(value) << offset);
    }
    bool operator<(const Array& other) const {
        for (size_t i = 0; i < ARRAY_LENGTH; ++i)
            if (data[i] != other.data[i]) return data[i] < other.data[i];
        return false;
    }
    bool operator==(const Array& other) const {
        for (size_t i = 0; i < ARRAY_LENGTH; ++i)
            if (data[i] != other.data[i]) return false;
        return true;
    }
    bool operator!=(const Array& other) const { return !(*this == other); }
    friend std::ostream& operator<<(std::ostream& os, const Array& array) {
        for (size_t i = 0; i < LENGTH; ++i) {
            if (i) os << ' ';
            os << array[i];
        }
        return os;
    }
    friend std::istream& operator>>(std::istream& is, Proxy proxy) {
        uint32_t value;
        is >> value;
        proxy = value;
        return is;
    }
};

using State = Array<15, 16>;

void print(State state) {
    rep(i, 4) {
        rep(j, 4) cout << "123456789ABCDEF-"[state[i * 4 + j]] << " ";
        cout << endl;
    }
}

vector<State> next_states(const State& state) {
    vector<State> states;
    states.reserve(4);
    const int dx[] = {0, 1, 0, -1};
    const int dy[] = {1, 0, -1, 0};
    int hole = 0;
    rep(i, 16) {
        if (state[i] == 15) {
            hole = i;
            break;
        }
    }
    int y = hole / 4, x = hole % 4;
    rep(k, 4) {
        int ny = y + dy[k], nx = x + dx[k];
        if (ny < 0 || ny >= 4 || nx < 0 || nx >= 4) continue;
        State new_state = state;
        new_state[hole] = state[ny * 4 + nx];
        new_state[ny * 4 + nx] = 15;
        states.emplace_back(new_state);
    }
    return states;
}

short h_star(const State& state) {
    short h = 0;
    rep(i, 16) {
        if (state[i] == 15) continue;
        int y = i / 4, x = i % 4;
        int yy = (state[i]) / 4, xx = (state[i]) % 4;
        h += abs(y - yy) + abs(x - xx);
    }
    return h;
}

template <typename State, typename Cost, vector<State> (*next_states)(const State& state),
          Cost (*h_star)(const State& state)>
vector<State> a_star(State start, State goal) {
    typedef pair<Cost, State> CS;
    priority_queue<CS, vector<CS>, greater<CS>> pq;
    map<State, Cost> dist;
    map<State, State> prv;
    dist[start] = h_star(start);
    pq.emplace(make_pair(h_star(start), start));
    while (!pq.empty()) {
        CS u = pq.top();
        Cost curDist = u.first;
        State curState = u.second;
        pq.pop();
        if (curState == goal) {
            break;
        }
        vector<State> nxt = next_states(curState);
        Cost h = h_star(curState);
        Cost fn = dist[curState];
        Cost gn = fn - h;
        if (curDist > fn) continue;
        if (curDist > 45) return {};

        const int dx[] = {0, 0, 1, -1};
        const int dy[] = {1, -1, 0, 0};
        for (size_t i = 0; i < nxt.size(); ++i) {
            State nxtState = nxt[i];
            Cost cost = 1;
            Cost nxtDist = gn + h_star(nxtState) + cost;
            if (dist.count(nxtState) == 0 || dist[nxtState] > nxtDist) {
                dist[nxtState] = nxtDist;
                prv[nxtState] = curState;
                pq.emplace(make_pair(nxtDist, nxtState));
            }
        }
    }
    vector<State> path;
    State cur = goal;
    while (cur != start) {
        path.emplace_back(cur);
        cur = prv[cur];
    }
    path.emplace_back(start);
    reverse(path.begin(), path.end());
    return path;
};

int main() {
    int t;
    cin >> t;
    while (t--) {
        State start, goal;
        rep(i, 16) cin >> (start[i]);
        rep(i, 16) {
            if (start[i] == 0) start[i] = 15;
            else start[i] = start[i] - 1;
        }
        rep(i, 16) goal[i] = i;

        // 順列の偶奇と空白マスの位置の偶奇が一致していれば解ける
        int inv = 0;
        rep(i, 16) rep2(j, i + 1, 16) {
            if (start[i] > start[j]) inv++;
        }
        int vy = -1, vx = -1;
        rep(i, 16) if (start[i] == 15) vy = i / 4, vx = i % 4;

        if (inv % 2 != (abs(vy - 3) + abs(vx - 3)) % 2) {
            cout << "This puzzle is not solvable." << endl;
            continue;
        }

        if (start == goal) {
            cout << endl;
            continue;
        }

        vector<State> path = a_star<State, short, next_states, h_star>(start, goal);
        if (path.empty()) {
            cout << "This puzzle is not solvable." << endl;
        } else {
            int prvY = -1, prvX = -1;
            rep(i, path.size()) {
                rep(y, 4) {
                    rep(x, 4) {
                        if (path[i][y * 4 + x] == 15) {
                            if (prvY != -1) {
                                if (prvY == y) {
                                    if (prvX < x) {
                                        cout << "R";
                                    } else {
                                        cout << "L";
                                    }
                                } else {
                                    if (prvY < y) {
                                        cout << "D";
                                    } else {
                                        cout << "U";
                                    }
                                }
                            }
                            prvY = y;
                            prvX = x;
                        }
                    }
                }
            }
            cout << endl;
        }
    }
}
```