---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:heading -->## はじめに

「プログラミングコンテスト攻略のためのアルゴリズムとデータ構造」の各問題を Python を用いて解いていくシリーズです。

[このシリーズについて](https://lonely-journalclub.com/algorithm/post-255/)

## 問題

> 次のプログラムは、[挿入ソート](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_1_A)を応用して n 個の整数を含む数列 A を昇順に整列するプログラムです。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_D](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_D)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">insertionSort(A, n, g)
      for i = g to n-1
          v = A[i]
          j = i - g
          while j >= 0 && A[j] > v
              A[j+g] = A[j]
              j = j - g
              cnt++
          A[j+g] = v

shellSort(A, n)
     cnt = 0
     m = ?
     G[] = {?, ?,..., ?}
     for i = 0 to m-1
         insertionSort(A, n, G[i])
```

> shellSort(A, n) は、一定の間隔 g だけ離れた要素のみを対象とした挿入ソートである insertionSort(A, n, g) を、最初は大きい値から g を狭めながら繰り返します。これをシェルソートと言います。
>
> 上の疑似コードの ? を埋めてこのプログラムを完成させてください。n と数列 A が与えられるので、疑似コード中の m、m 個の整数 Gi(i=0,1,...,m−1)、入力 A を昇順にした列を出力するプログラムを作成してください。ただし、出力は以下の条件を満 たす必要があります。
>
> https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_D

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import time
import random


def insertion_sort(n_list, g):
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更されてしまう
    sorted_list = list(n_list)

    # リストのg番目からまず親ポインタを動かす
    for i in range(g, len(sorted_list)):
        # 今親ポインタがある要素を取得
        pointed_value = sorted_list[i]
        # 子ポインタ(親ポインタから左端に向かってg分離れた場所からg分ずつ左端に向かって動くポインタ)のインデックスを初期化
        j = i - g
        # 子ポインタのある場所の要素と親ポインタのある場所の要素を比較し、子ポインタのある場所の要素の方が大きかったら、親ポインタのある場所を子ポインタの要素で置き換える
        # 子ポインタのある場所の要素が親ポインタのある場所の要素よりも小さくなるか、小ポインタのインデックスが0を下回るまでループを回す。
        while j >= 0 and (sorted_list[j] > pointed_value):
            sorted_list[j + g] = sorted_list[j]
            # Pythonでは--のような演算子は使えない
            j -= g

        #最後に要素間の大きさの比較を行った子ポインタがある場所に親ポインタの要素を入れる
        sorted_list[j + g] = pointed_value

    return sorted_list


def shell_sort(n_list):
    start = time.time()
    g_list = []
    h = 1
    while h <= len(n_list):
        g_list.append(h)
        h = 3 * h + 1
    g_list.reverse()
    for g in g_list:
        sorted_list = insertion_sort(n_list, g)
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list
```

これまでの研究から、ソートを行う間隔を与える g は

gn+1 = 3gn+1 の数列(例えば、1,4,13,40,121...)

とすると計算量が O(N1.25)となり、良いとされているらしい。
