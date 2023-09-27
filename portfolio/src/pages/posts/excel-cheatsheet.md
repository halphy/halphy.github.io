---
layout: '../../layouts/BlogPostLayout.astro'
title: '競技Excelチートシート'
pubDate: 2022-09-27
description: 'Excelの関数を使った競技プログラミングもどき「競技Excel」のためのチートシート'
author: 'halphy'
tags: ["Excel"]
---

## 文字列操作系
### 連結
文字列の連結には演算子`&`を用いる．
```
= "hoge" & "fuga"   // hogefuga
```

### 部分文字列
文字列`str`の`i`文字目から`l`文字分を取得するには`MID(str, i, l)`と書く．
```
= MID("hoge", 2, 2)   // og
```

### 分割
文字列を区切り文字で分割して得られる文字列の配列を取得するには`TEXTSPLIT`関数を用いる．
```
= TEXTSPLIT("hoge,fuga,piyo", ",")   // ["hoge", "fuga", "piyo"]
```

## 配列
### 連番の生成
`SEQUENCE(行数, 列数, 初期値, 増分)`で等差数列を生成することができる．
```
= SEQUENCE(3)   // [[1], [2], [3]]
```
```
= SEQUENCE(2, 3, 0, 5)   // [[0, 5, 10], [15, 20, 25]]
```

### 配列の生成
`m`行`n`列の配列を生成するには`MAKEARRAY`関数を用いる．第3引数にラムダ式を指定することで，Pythonでいうリスト内包表記のように定義することできる．
```
= MAKEARRAY(2, 3, LAMBDA(i, j, i + j))

/*
[ [2, 3, 4],
  [3, 4, 5] ]
*/
```

## 繰り返し
### 擬似`for`文
`SEQUENCE`関数を用いて生成した連番`[[1], ..., [N]]`の配列に`MAP`関数を適用することで，インデックス`i (i = 1, ..., N)`に対する処理を記述できる．
```
= MAP(SEQUENCE(5), LAMBDA(i, 2*i) )   // [[2], [4], [6], [8], [10]]
```

### 累積値
`REDUCE`関数を用いると，配列に対して累積値を得ることができる．

例えば，
```
LAMBDA(ARR, REDUCE(0, ARR, LAMBDA(RES, VAL, RES + VAL)))
```
は以下のPythonコードと等価である．
```python
def f(ARR):
    RES = 0

    for VAL in ARRAY:
        RES = RES + VAL
    
    return RES
```

累積値の推移をまとめて配列として得るには，`SCAN`関数を用いる．

例えば，
```
LAMBDA(ARR, SCAN(0, ARR, LAMBDA(RES, VAL, RES + VAL)))
```
は以下のPythonコードと等価である．
```python
def f(ARR):
    RES = 0
    SCN = []

    for VAL in ARRAY:
        RES = RES + VAL
        SCN.append(RES)
    
    return SCN
```
