---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

> **総当たり**
> 長さ n の数列 A と整数 m に対して、A の要素の中のいくつかの要素を足しあわせて m が作れるかどうかを判定するプログラムを作成してください。A の各要素は１度だけ使うことができます。
> 数列 A が与えられたうえで、質問として q 個の mi が与えれるので、それぞれについて yes または no と出力してください。
>
> **入力**
> １行目に n、２行目に A を表す n 個の整数、３行目に q、４行目に q 個の整数 mi が与えられます。
>
> **出力**
> 各質問について A の要素を足しあわせて mi を作ることができれば yes と、できなければ no と出力してください。
>
> **制約**
> n≤20
> q≤200
> 1≤A の要素 ≤2,000
> 1≤mi≤2,000
>
> [http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_5_A&lang=ja](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_5_A&lang=ja)

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def solve(i, target):
    global input_list
    if target == 0:
        return True
    if i >= len(input_list):
        return False
    res = solve(i + 1, target) or solve(i + 1, target - input_list[i])
    return res


target = 5
input_list = [1, 5, 7, 10, 21]
solve(0, target)

target = 4
input_list = [2, 4, 17, 8]
solve(0, target)

target = 25
input_list = [2, 4, 17, 8]
solve(0, target)
```

solve(i,target)を「input 配列の i 番目以降の要素を使って target を作れる場合 True を返却する」メソッドとした場合、今回の問題設定では、solve(0,target)が True になるかどうかを調べることが課題となる。

そのため、まず solve(0,target)を部分問題に分割し、solve(i+1,target)と solve(i+1,target - input 配列[i])のどちらかが True であれば、True が返却されるとする。
i をどんどん増やしていき、どこかのプロセスの分岐で target - input 配列[i]=0 となれば、返却値は True になり、target を満たす組み合わせがあることがわかる。
