---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:paragraph -->**「プログラミングコンテスト攻略のためのアルゴリズムとデータ構造」**

https://www.amazon.co.jp/dp/B00U5MVXZO/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1

の「6.2 全探索」で登場する再帰関数を使った組み合わせ列挙問題を Python で実装してみた。

## 問題

以下の input/output を実現する関数を実装する。

input:組み合わせを列挙したい配列
output:input 配列の中で組み合わせに含まれる要素の番地が 1,含まれない要素の番地が 0 になった配列群を格納したリスト

### コード

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def make_combination(input_list):
    def rec(n):
        nonlocal S_list
        nonlocal combinations_list
        if n == len(input_list):
            print(S_list)
            combinations_list.append(list(S_list))
            return

        #①の分岐生成
        rec(n + 1)

        #②の分岐生成
        S_list[n] = 1
        rec(n + 1)

        S_list[n] = 0

    combinations_list = []
    S_list = [0 for i in range(len(input_list))]
    rec(0)
    return combinations_list

input_list = [1, 5, 7]
combibations_list = make_combination(input_list)
print(combibations_list)
#[[0, 0, 0], [0, 0, 1], [0, 1, 0], [0, 1, 1], [1, 0, 0], [1, 0, 1], [1, 1, 0], [1, 1, 1]]
```

## 解説になってない解説

最初に組み合わせを列挙したい配列と要素数が同じで全ての要素が 0 の配列を作成し、それに再帰関数を用いて次々変更を加えていって全ての組合わせを列挙し、それらを conbinations_list に格納して返却するというのが全体の流れ。

キモとなる再帰関数がややこしい。一応処理の流れっぽいものを図にしてみたが、分かりづらい。

コード中で ① の分岐生成とコメントされている箇所で生成されたプロセスが下図の ①、② の分岐生成とコメントがされている箇所で生成されたプロセスが下図の ② に該当する。
水色の[0,0,1]といった配列は再帰関数実行実行時の S_list。
赤字の配列は結果として出力される組み合わせパターン。

![](https://lonely-journalclub.com/wp-content/uploads/2020/03/-22-3-e1585026898597-763x1024.jpg)
