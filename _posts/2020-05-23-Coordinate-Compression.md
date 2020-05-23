---
layout: post
title: 座標圧縮
---

# 座標圧縮

# 問題
- [http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_4_A](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_4_A)
- [https://atcoder.jp/contests/abc168/tasks/abc168_f](https://atcoder.jp/contests/abc168/tasks/abc168_f)
- [https://atcoder.jp/contests/joi2008ho/tasks/joi2008ho_e](https://atcoder.jp/contests/joi2008ho/tasks/joi2008ho_e)
- [https://atcoder.jp/contests/joi2013yo/tasks/joi2013yo_e](https://atcoder.jp/contests/joi2013yo/tasks/joi2013yo_e)
- [https://atcoder.jp/contests/abc008/tasks/abc008_4](https://atcoder.jp/contests/abc008/tasks/abc008_4)
- [https://atcoder.jp/contests/apc001/tasks/apc001_i](https://atcoder.jp/contests/apc001/tasks/apc001_i)

# AOJ DSL_4_A Union of Rectangles
[http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_4_A](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=DSL_4_A)

## 概要
長方形の左下/右上の点の座標が与えられるとき、1つ以上の長方形で囲まれている領域の面積を求める問題。

## 解法

座標圧縮 + いもす法

点の座標の範囲が大きい($$-10^9$$~$$10^9$$)ので、そのままグリッドをつくることはできない。そこで、座標圧縮をおこなうと、$$2\times\lvert\text{点の集合}\rvert$$のグリッドに納められるので、いもす法を使って面積を求められる。

- 長方形をなす線分をグリッド上で扱いたいので、線分を幅0としてグリッド上で扱う
- 偶数番目のグリッドは幅0とする

### 実装

```
int main() {
    i64 N; cin >> N;
    vector<i64> X1(N);
    vector<i64> Y1(N);
    vector<i64> X2(N);
    vector<i64> Y2(N);

    // --> compression
    vector<i64> xs;
    vector<i64> ys;
    REP(i, N) {
        cin >> X1[i] >> Y1[i] >> X2[i] >> Y2[i];
        xs.pb(X1[i]), xs.pb(X2[i]);
        ys.pb(Y1[i]), ys.pb(Y2[i]);
    }

    sort(xs.begin(), xs.end());
    UNIQUE(xs);
    sort(ys.begin(), ys.end());
    UNIQUE(ys);
    // compression <--

    // --> color (with imos method) inside rectangles in compressed grid
    int w = xs.size() * 2; // add 0-width row/col
    int h = ys.size() * 2;
    vector<vector<int>> G(w, vector<int>(h, 0));
    REP(i, N) {
        int x1 = (lower_bound(xs.begin(), xs.end(), X1[i]) - xs.begin()) * 2;
        int x2 = (lower_bound(xs.begin(), xs.end(), X2[i]) - xs.begin()) * 2;
        int y1 = (lower_bound(ys.begin(), ys.end(), Y1[i]) - ys.begin()) * 2;
        int y2 = (lower_bound(ys.begin(), ys.end(), Y2[i]) - ys.begin()) * 2;
        G[x1][y1]++;
        G[x2 + 1][y2 + 1]++;
        G[x1][y2 + 1]--;
        G[x2 + 1][y1]--;
    }
    REP(x, w) REP(y, h - 1) G[x][y + 1] += G[x][y];
    REP(y, h) REP(x, w - 1) G[x + 1][y] += G[x][y];
    // color inside rectangles in compressed grid <--

    // --> calculate total area in original grid
    i64 ans = 0;
    REP(x, w) {
        REP(y, h) {
            if (G[x][y] < 1) continue;
            // ignore even row/col of width 0
            if (x % 2 == 0 || y % 2 == 0) continue;
            ans += (xs[x/2 + 1] - xs[x/2]) * (ys[y/2 + 1] - ys[y/2]);
        }
    }
    cout << ans << endl;
    // calculate total area in original grid <--

    return 0;
}
```
