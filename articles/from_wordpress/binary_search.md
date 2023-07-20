---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題 > n個の整数を含む数列 Sと、q個の異なる整数を含む数列 Tを読み込み、Tに含まれる整数の中で Sに含まれるものの個数 C出力するプログラムを作成してください。 > ただし、Sの要素は昇順に整列されています。 > > [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_B](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_B) 線形探索と同じ問題を二分探索を用いて解いてみます。 ## 解法 ``` ```

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import random
import time


def binary_search(x, n_list):
    left = 0
    right = len(n_list) - 1
    while left  x:
            right = mid - 1
        else:
            left = mid + 1
    return False


S = [random.randint(1, 10000) for i in range(1000000)]
T = set([random.randint(1, 10000) for i in range(100)])


# 二分探索
count = 0
start1 = time.time()
for t in T:
    if binary_search(t, S):
        count += 1
print(elapsed_time:{0}.format(time.time() - start1))
print(count)
```

実行時間は**約 0.0008 秒**で、同じテストに約 0.03 秒かかっていた線形探索と比べるとだいぶ早くなっていました。

理論的には、線形探索で O(n)であった計算量の処理を O(log2n)で出来ることになるので、事前に要素のソートさえできれば、積極的に使ったほうがいいアルゴリズムと言えそうです。

```

```
