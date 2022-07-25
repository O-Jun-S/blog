---
title: "ABC261 D"
date: 2022-07-25T17:37:09+09:00
tags: ["競プロ", "ABC"]
keywords: ["dp"]
description: "ABC 261 D問題の解説"
draft: false
---

## 問題文
[問題のリンク](https://atcoder.jp/contests/abc261/tasks/abc261_d)

N回のコイントスを行って、表が出たらお金がもらえます。
しかし、表を出し続ければよいというわけではありません。
この問題を難しくしているのが、連続で表を出すことによるボーナスです。
なぜなら、例えば「2回連続表が出た場合」のボーナスがとても高い場合を考えます。
このとき、10回連続表を出すよりも、裏を出してリセットし、先ほどのボーナスを取り続けたほうが儲かるかもしれません。
コイントスを行って得たお金の最大値を求めるのがこの問題です。

## 解法
結論から言うと、解法はdp(動的計画法、Dynamic Programming)です。
dpの特徴は、「前の計算結果を利用して次の結果を出す」ことです。
dpについて知らない方は、[この記事](https://qiita.com/drken/items/dc53c683d6de8aeacf5a)を見ると分かりやすいかもしれないです。
dpを使うという判断の理由は、
- 前の計算からの結果が利用できそうである。
- 最適な行動をした場合の最大値を求める問題である。

などです。

さて、dpと言えば、考えるのはdpテーブルです。僕は以下のように定義しました。
$$
dp[i][j] = i番目のコイントスで、連続でj回表になっているとき、得られるお金の最大値。
$$
ただし、iは1-indexed。

次に考えるべきは、初期値と漸化式です。僕は次のように設定しました。
- 初期値 $dp[0][0] = 0$
- 表が出た場合 $dp[i+1][j+1] = \max(dp[i+1][j+1], dp[i][j] + X[i] + bonus[j+1])$
- 裏が出た場合 $dp[i+1][0] = \max(dp[i+1][0], dp[i][j])$

ただし、bonus[j]は、j回連続表が出た時のボーナス(ボーナスが出ない時は0)。

答えは、次のように選べばよいです。
$$
ans = \max_{0 \le j \le n}(dp[n][j])
$$

よって、$O(N^2)$でこの問題が解けました。
$1 \le n \le 5000$より、十分間に合います。

僕が提出したコードは以下です。

```cpp
#include<bits/stdc++.h>
using namespace std;
#define rep(i, n) for(int i=0; i < (n); i++)
using ll = long long;
template<class T> inline bool chmax(T& a, T b) { if (a < b) { a = b; return 1; } return 0; }
template<class T> inline bool chmin(T& a, T b) { if (a > b) { a = b; return 1; } return 0; }

int main(void) {
    int n, m;
    cin >> n >> m;

    vector<int> x(n);
    rep(i, n) cin >> x[i];

    vector<vector<ll>> dp(5500, vector<ll>(5500, -1));

    map<ll, ll> bonus;
    rep(i, m) {
        ll ci, yi;
        cin >> ci >> yi;
        bonus[ci] = yi;
    }

    dp[0][0] = 0;
    rep(i, n) {
        rep(j, 5500) {
            if(dp[i][j] == -1) continue;
            chmax(dp[i + 1][0], dp[i][j]);
            chmax(dp[i + 1][j + 1], dp[i][j] + x[i] + bonus[j + 1]);
        }
    }

    ll ans = 0;
    rep(j, 5500) {
        chmax(ans, dp[n][j]);
    }

    cout << ans << endl;
    return 0;
}
```

* 5500という値は僕が「nの上限が5000だから適当に5500でループすればええやろ」と思って書いた値です。
適宜、nやn+1に変えてもらえればより綺麗な解答になると思います。

## 余談
dpを書くとき、$dp[i][j] = \max(dp[i][j], \ldots)$という式を書かなければなりません。
しかし、少し冗長です。
そこで、僕はそれをシンプルに書けるchminとchmaxという関数をテンプレートとして持っています。
[参考](https://qiita.com/drken/items/dc53c683d6de8aeacf5a)

実装は以下です。
```cpp
template<class T> inline bool chmax(T& a, T b) { if (a < b) { a = b; return 1; } return 0; }
template<class T> inline bool chmin(T& a, T b) { if (a > b) { a = b; return 1; } return 0; }
```
知らなかった方はコピペして使ってみてはいかがでしょうか。

