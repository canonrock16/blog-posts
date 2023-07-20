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

> トランプのカードを整列しましょう。ここでは、４つの絵柄(S, H, C, D)と９つの数字(1, 2, ..., 9)から構成される計 36 枚のカードを用います。例えば、ハートの 8 は H8、ダイヤの 1 は D1 と表します。
> バブルソート及び選択ソートのアルゴリズムを用いて、与えられた _N_ 枚のカードをそれらの数字を基準に昇順に整列するプログラムを作成してください。アルゴリズムはそれぞれ以下に示す疑似コードに従うものとします。数列の要素は 0 オリジンで記述されています。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_C](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_C)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">BubbleSort(C, N)
   for i = 0 to N-1
     for j = N-1 downto i+1
       if C[j].value < C[j-1].value
         C[j] と C[j-1] を交換

SelectionSort(C, N)
  for i = 0 to N-1
    minj = i
    for j = i to N-1
      if C[j].value < C[minj].value
        minj = j
    C[i] と C[minj] を交換
```

> また、各アルゴリズムについて、与えられた入力に対して安定な出力を行っているか報告してください。ここでは、同じ数字を持つカードが複数ある場合それらが入力に出現する順序で出力されることを、「安定な出力」と呼ぶことにします。（※常に安定な出力を行うソートのアルゴリズムを安定なソートアルゴリズムと言います。）
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_C](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_C)

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import random
import time


def bubble_sort(n_list):
    start = time.time()
    swap_count = 0
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)
    for i in range(len(sorted_list)):
        for j in reversed(range(i + 1, len(sorted_list))):
            if sorted_list[j][1] < sorted_list[j - 1][1]:
                swap_count += 1
                tmp = sorted_list[j]
                sorted_list[j] = sorted_list[j - 1]
                sorted_list[j - 1] = tmp

    print(swap_count, swap_count)
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list


def selection_sort(n_list):
    start = time.time()
    swap_count = 0
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)

    for i in range(len(sorted_list)):
        min_j = i
        for j in range(i, len(sorted_list)):
            if sorted_list[j][1] < sorted_list[min_j][1]:
                min_j = j
        if i != min_j:
            swap_count += 1
            tmp = sorted_list[i]
            sorted_list[i] = sorted_list[min_j]
            sorted_list[min_j] = tmp

    print(swap_count, swap_count)
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list
```

数字を基準に整列せよという問題なので、n_list[j][1] < n_list[min_j][1]の所で[1]を付けています。それ以外は 3.3,3.4 で作成したコードと同じです。

また、ソートが安定かどうかを判定する計算量 O(n4)の関数を作成しました。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def is_stable(in_list, out_list):
    start = time.time()
    for i in range(len(in_list)):
        for j in range(i + 1, len(in_list)):
            for a in range(len(in_list)):
                for b in range(a + 1, len(in_list)):
                    if (
                        # in_list内に数字部分が同じ値があって,
                        in_list[i][1] == in_list[j][1]
                        # in_list[i]の値とin_list[j]の値の順番がout_list内で逆転していたら
                        and in_list[i] == out_list[b]
                        and in_list[j] == out_list[a]
                    ):
                        # それは安定でないソートである
                        print(False!!)
                        print(i, i)
                        print(j, j)
                        print(a, a)
                        print(b, b)
                        print(in_list_i, in_list[i])
                        print(in_list_j, in_list[j])
                        print(out_list_b, out_list[b])
                        print(out_list_a, out_list[a])
                        print(-------------)
                        return False
    print(elapsed_time:{0}.format(time.time() - start))
    return True
```

実装を見ただけではなぜこれで安定かどうかを判定できるのか分かりづらいかも知れないので、軽くこのソートが安定かどうかを判定する関数について説明します。

「ソートが安定である」とは、ソートする前と後で、ソートに使ったキー以外の要素の順番が変わらないことを意味します。これをチェックしたいので、ソートされる配列の中で数字部分が同じである要素同士の順序が同じであるかどうかを調べます。
具体的にどのように調べるかというと、常に i < j である i と j,常に a < b である a と b を用意し、in_list[i]==out_list[b]、in_list[j]==out_list[a]が成立してしまっている組み合わせが無いかを調べます。in_list の中では必ず j の要素が後に来るはずなのに、out_list の中では先に来てしまっている場合は安定でないと判定するということです。

## テストと実行速度測定

### テストケース 1.ランダム状態からのソート

以下を実行。アルファベットと数字の組み合わせリストを作成し、random.shuffle()でリストをランダム化。それをバブルソートと選択ソートにかけ、安定であるかどうかをチェックする。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def make_card_list():
    n_list = []
    for string in [S, H, C, D]:
        for num in range(1, 10):
            n_list.append(string + str(num))
    random.shuffle(n_list)
    return n_list

n_list = make_card_list()
# バブルソートのチェック
orig_sort = bubble_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)


# 選択ソートのチェック
orig_sort = selection_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)
```

#### 結果

**バブルソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-30-1024x138.png)バブルソートは安定なソートであることが確認できた。

**選択ソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-31-1024x225.png)選択ソートは離れた位置にある数同士を交換するため、安定なソートではない。この例でも安定なソート結果にはならなかった。

### テストケース 2.降順状態からのソート

ケース 1 と同様の手法でリストを作成し、.sort(reverse=True)で降順にソートをかけた。それをバブルソートと選択ソートにかけ、安定であるかどうかをチェックする。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = make_card_list()
n_list.sort(reverse=True)

# バブルソートのチェック
orig_sort = bubble_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)

# 選択ソートのチェック
orig_sort = selection_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)
```

#### 結果

**バブルソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-32-1024x151.png)やはり安定な結果となった。

**選択ソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-33-1024x223.png)安定ではない。

### テストケース 3.昇順状態からのソート

ケース 1 と同様の手法でリストを作成し、.sort()で昇順にソートをかけた。それをバブルソートと選択ソートにかけ、安定であるかどうかをチェックする。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = make_card_list()
n_list.sort()

# バブルソートのチェック
orig_sort = bubble_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)

# 選択ソートのチェック
orig_sort = selection_sort(n_list)
print(n_list)
print(orig_sort)
is_stable(n_list, orig_sort)
```

#### 結果

**バブルソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-34-1024x148.png)**選択ソート**

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-35-1024x224.png)バブルソートは安定、選択ソートは安定でない結果になった。

## まとめ

バブルソートは常に安定なソートであるが、選択ソートはそうではない。

#### 感想

安定であるか否かをチェックする関数が計算量が多く実行に時間がかかってしまう。バブルソートが常に安定なソートであることを踏まえると、バブルソートの結果との比較で安定であるかどうかチェックしたほうがいいかも知れない。
