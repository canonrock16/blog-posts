---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

> 重さがそれぞれ wi(i=0,1,...n−1)の n 個の荷物が、ベルトコンベアから順番に流れてきます。これらの荷物を k 台のトラックに積みます。各トラックには連続する 0 個以上の荷物を積むことができますが、それらの重さの和がトラックの最大積載量 P を超えてはなりません。最大積載量 P はすべてのトラックで共通です。
> n、k、wi が与えられるので、すべての荷物を積むために必要な最大積載量 P の最小値を求めるプログラムを作成してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_D](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_D)

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import random
import time


def isloadable(p, k, w_list):
    if sum(w_list) <= p * k:
        return True
    else:
        return False


def binary_solver(k, w_list):
    start = time.time()
    left = 0
    right = 10000
    while left < right:
        mid = round((left + right) / 2)
        if isloadable(mid, k, w_list):
            right = mid
        else:
            left = mid + 1
        if right - left == 1:
            if isloadable(left, k, w_list):
                print(elapsed_time:{0}.format(time.time() - start))
                return left
            else:
                print(elapsed_time:{0}.format(time.time() - start))
                return right

    print(elapsed_time:{0}.format(time.time() - start))
    return right


def linear_solver(k, w_list):
    start = time.time()
    p = 0
    while not (isloadable(p, k, w_list)):
        p += 1
    print(elapsed_time:{0}.format(time.time() - start))
    return p


w_list = [random.randint(1, 100) for i in range(1000)]
k = 10
print(binary_solver(k, w_list))
print(linear_solver(k, w_list))
```

線形探索で解く方法と二分探索で解く方法の二種類を書いてみました。
処理時間は**線形探索が約 0.044 秒**、**二分探索が約 0.0001 秒**とやはり二分探索の方が早い結果となりました。
