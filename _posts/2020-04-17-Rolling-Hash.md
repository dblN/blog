---
layout: post
title: Rolling Hash
---


# ローリングハッシュ
文字列を効率的にハッシュする手法。ある文字列Sに文字列Tが何個含まれているかを数える、文字列TのS内での位置を調べる場合などに使われる。
アルゴリズムは単純で、累積和の考えに近い。

## 参考
実装は以下を参考にした。

- [http://algoogle.hadrori.jp/algorithm/rolling-hash.html](http://algoogle.hadrori.jp/algorithm/rolling-hash.html)
- [https://ei1333.github.io/luzhiled/snippets/string/rolling-hash.html](https://ei1333.github.io/luzhiled/snippets/string/rolling-hash.html)
- [https://qiita.com/keymoon/items/11fac5627672a6d6a9f6](https://qiita.com/keymoon/items/11fac5627672a6d6a9f6)

  - 衝突を起こさないようにするために、どのようにmodと基数を選ぶべきかにおいて大変参考になった
- [https://www.slideshare.net/nagisaeto/rolling-hash-149990902](https://www.slideshare.net/nagisaeto/rolling-hash-149990902)

  - 適当にローリングハッシュを実装すると、簡単に衝突を起こせるという話

## 問題
- [https://atcoder.jp/contests/abc141/tasks/abc141_e](https://atcoder.jp/contests/abc141/tasks/abc141_e)
- [http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_14_B&lang=jp](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_14_B&lang=jp)

# 実装
templateを使っているが、modは$$2^{61}-1$$を前提とする。
基数は実行時に、文字集合より大きな数をランダムに選んで初期化する。衝突が起こる場合は、基数を複数用意して対処する。

{% highlight cpp %}
#define MOD 2305843009213693951 // 2^61 - 1

template<u64 mod>
struct RollingHash {
    i64 n;
    vector<u64> hashed, power;

    RollingHash(const string &s, u64 base = 1'000'007) {
        n = s.length();
        hashed.assign(n + 1, 0);
        power.assign(n + 1, 0);
        hashed[0] = 0;
        power[0] = 1;
        REP(i, n) {
            power[i + 1] = calc_mod(mul(power[i], base));
            hashed[i + 1] = calc_mod(mul(hashed[i], base) + s[i]);
        }
    }

    u64 hash(i64 l, i64 r) {
        return calc_mod(hashed[r] + 3 * mod - mul(hashed[l], power[r - l]));
    }

    bool match(i64 l1, i64 r1, i64 l2, i64 r2) {
        return hash(l1, r1) == hash(l2, r2);
    }

private:
    const u64 MASK31 = (1ULL << 31) - 1;
    const u64 MASK30 = (1ULL << 30) - 1;
    const u64 MASK61 = (1ULL << 61) - 1;

    u64 mul(u64 a, u64 b) {
        u64 ah = a >> 31;
        u64 al = a & MASK31;
        u64 bh = b >> 31;
        u64 bl = b & MASK31;
        u64 mid = ah * bl + al * bh;
        u64 midh = mid >> 30;
        u64 midl = mid & MASK30;
        return ah * bh * 2 + midh + (midl << 31) + al * bl;
    }

    u64 calc_mod(u64 x) {
        u64 xh = x >> 61;
        u64 xl = x & MASK61;
        u64 res = xh + xl;
        if (res >= mod) res -= mod;
        return res;
    }
};
{% endhighlight %}