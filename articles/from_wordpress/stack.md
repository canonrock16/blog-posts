---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

> 逆ポーランド記法は、演算子をオペランドの後に記述する数式やプログラムを記述する記法です。例えば、一般的な中間記法で記述された数式 (1+2)_(5+4) は、逆ポーランド記法では 1 2 + 5 4 + _ と記述されます。逆ポーランド記法では、中間記法で必要とした括弧が不要である、というメリットがあります。
> 逆ポーランド記法で与えられた数式の計算結果を出力してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/3/ALDS1_3_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/3/ALDS1_3_A)

スタック構造にデータを格納し、逆ポーランド記法の数式の計算を行う問題です。今回の実装では Python のリストを用いてスタック構造を実現します。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import sys


class stack:
    def __init__(self, MAX=10):
        self.MAX = MAX
        self.S = []

    def __isEmpty__(self,):
        return len(self.S) == 0

    def __isFull__(self,):
        return len(self.S) >= self.MAX - 1

    def push(self, x):
        if self.__isFull__():
            sys.exit(overflow)
        self.S.append(x)

    def pop(self):
        if self.__isEmpty__():
            sys.exit(underflow)
        return self.S.pop()


input_list = [1, 2, +, 3, 4, -, *]
s = stack(100)
for x in input_list:
    if str(x).isdecimal():
        s.push(x)
    else:
        b = s.pop()
        a = s.pop()
        if x == +:
            s.push(a + b)
        elif x == -:
            s.push(a - b)
        elif x == *:
            s.push(a * b)
print(s.pop())
```
