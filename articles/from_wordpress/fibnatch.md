---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

[AIZU ONLINE JUDGE のこの問題を参照。](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_10_A&lang=ja)

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import time


def fibonacci(n):
    global f_memo
    if n == 0 or n == 1:
        f_memo[n] = 1
    if n in f_memo.keys():
        return f_memo[n]
    f_memo[n] = fibonacci(n - 2) + fibonacci(n - 1)
    return f_memo[n]


def roop_fibonacci(n):
    global f_memo
    f_memo[0] = 1
    f_memo[1] = 1

    for i in range(2, n + 1):
        f_memo[i] = f_memo[i - 2] + f_memo[i - 1]

    return f_memo[n]


f_memo = {}
start = time.time()
fibonacci(1500)
print(elapsed_time:{0}.format(time.time() - start))

f_memo = {}
start = time.time()
roop_fibonacci(1500)
print(elapsed_time:{0}.format(time.time() - start))
```

トップダウン(フィボナッチ数列の n 番目の数値を算出するためのプロセスが最初に走り、そこから n-1 番目の数値の算出プロセス、n-2 番目の数値の算出プロセス...と処理していく方法)での実装を fibonacci,

ボトムアップ(フィボナッチ数列 2 番目の数値の算出から開始し、n 番目まで順に処理していく方式)での実装を roop_fibonacci とした。

それぞれの処理時間は以下の通りで、何回か試した傾向としてはボトムアップ式の方が早めではあるものの、大きな差はなかった。

#### トップダウン

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">elapsed_time:0.0033102035522460938
```

#### ボトムアップ

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">elapsed_time:0.0016460418701171875
```
