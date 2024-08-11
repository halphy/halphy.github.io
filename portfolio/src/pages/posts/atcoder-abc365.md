---
layout: '../../layouts/BlogPostLayout.astro'
title: 'AtCoder Beginner Contest 365に参加した'
pubDate: 2024-08-11
description: 'ABC365の参加記．'
author: 'halphy'
tags: ["競プロ", "アルゴ", "AtCoder"]
---

8月3日に開催された[トヨタ自動車プログラミングコンテスト2024#8（AtCoder Beginner Contest 365）](https://atcoder.jp/contests/abc365)に参加した．ABCDEの5完で509位．

## A - Leap Year
[問題文](https://atcoder.jp/contests/abc365/tasks/abc365_a)

西暦$Y$年の日数を答える問題．要するにうるう年かどうかが判定できればよく，うるう年の判定条件は問題文で与えられているので言われた通り実装する．

提出：https://atcoder.jp/contests/abc365/submissions/56238645

## B - Second Best
[問題文](https://atcoder.jp/contests/abc365/tasks/abc365_b)

長さ$N$の数列$(A_i)$が与えられるので，$A$の中で2番目に大きい要素のインデックスを求める問題．

Rustだと[`sorted_by_key`](https://docs.rs/itertools/latest/itertools/trait.Itertools.html#method.sorted_by_key)を使ってイテレータ`0..n`をソートするのが楽かも．

提出：https://atcoder.jp/contests/abc365/submissions/56238645

## C - Transportation Expenses
[問題文](https://atcoder.jp/contests/abc365/tasks/abc365_c)

整数$N, M$と数列$(A_i)$が与えられるとき，
$$
\sum_i \min(x, A_i)\leq M
$$
を満たす最大の整数$x(\geq 0)$を求める問題．ストーリーは「$1,\cdots, N$の$N$人に交通費$A_i$円を支給するとき，予算$M$円を超えないように1人あたりの支給上限額$x$をどこまで大きくできるか？」というもの．

まず，上限額なしでも予算内に収まる場合（$\sum_i A_i\leq M$のとき）は最大値が存在しない．そうでない場合は$0\leq x\leq M$の範囲に最大値が存在するので二分探索する．

二分探索パートは、判定関数`f`，探索範囲`[ok, ng)`（または`(ng, ok]`）を渡すようなライブラリを事前に作っておくとバグりにくい．なお探索範囲の下限・上限ではなく`true`側の端点・`false`側の端点を渡すのはいわゆる「めぐる式」の実装である（[出典](https://x.com/meguru_comp/status/697008509376835584)）．

提出：https://atcoder.jp/contests/abc365/submissions/56253079

## D - AtCoder Janken 3
[問題文](https://atcoder.jp/contests/abc365/tasks/abc365_d)

2人でじゃんけんを$N$回する．相手が出す手が分かっているとき，直前と同じ手を出すことはできないという条件のもとで自分は最大何回勝つことができるか？　という問題．

これは以下のような値を更新していくDPで解くことができる．

- ${\rm dp}[i][j]$：$i$番目の試合までみたとき，直前に出した手が$j$であるときの勝った回数の最大値

同じ手は連続して出せないので，$(i, j)$から遷移できるのは$(i + 1, k) \; (j\neq k)$である．

実装上は，`R`, `P`, `S`を$0,1,2$に対応させるなどすると勝利判定が少し楽になる．実際

- 手$x$が手$y$に勝つ $\quad\Longleftrightarrow\quad y+1\equiv x\pmod 3$

なので面倒な場合分けが最小限で済む．

提出：https://atcoder.jp/contests/abc365/submissions/56260604

## E - Xor Sigma Problem

[問題文](https://atcoder.jp/contests/abc365/tasks/abc365_e)

長さ$N$の整数列$(A_i)$が与えられるので，幅2以上のすべての区間に対する区間XORの総和

$$
\sum_{l+1<r} A_l\oplus \cdots\oplus A_{r-1}
$$

を求める問題．

定石に従い，桁ごとに考えて最後にそれぞれの項の和への寄与を足しあげることにする（$d$桁目には係数として$\times 2^d$がかかる）．bitwise XORの定義から，$A_l\oplus \cdots\oplus A_{r-1}$の$d$桁目というのは$A_l, \cdots, A_{r-1}$の$d$桁目に$1$が奇数個あれば$1$，偶数個あれば$0$になる．

よって，長さ$N$の01列の中で奇数個の$1$を含むような区間を数え上げればよく，これは累積和を${\rm mod} \;2$で構築し値が異なるペアを数えればOK（01列$x_1,\cdots, x_N$に対して，$[l, r)$が奇数個の$1$を含む $\Leftrightarrow$累積和を$s_i\equiv \sum_{j\in [0, i)}x_j$としたとき$s_r-s_l$が奇数$\Leftrightarrow\; s_l\not\equiv s_r\pmod 2$）．最後に幅1の区間を計算結果から除外することを忘れない．

提出：https://atcoder.jp/contests/abc365/submissions/56272194
