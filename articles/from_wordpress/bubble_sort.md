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

> バブルソートはその名前が表すように、泡（Bubble）が水面に上がっていくように配列の要素が動いていきます。バブルソートは次のようなアルゴリズムで数列を昇順に並び変えます。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_A)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">bubbleSort(A, N) // N 個の要素を含む 0-オリジンの配列 A
   flag = 1 // 逆の隣接要素が存在する
   while flag
     flag = 0
     for j が N-1 から 1 まで
       if A[j] < A[j-1]
         A[j] と A[j-1] を交換
         flag = 1
```

> 数列 _A_ を読み込み、バブルソートで昇順に並び変え出力するプログラムを作成してください。また、バブルソートで行われた要素の交換回数も報告してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_A)

ソートアルゴリズムの一つ、バブルソートです。右からポインタを動かしていって、左隣の数字の方が大きかったら今ポインタがあたっている数と入れ替えるという操作をソート完了まで続けるソート手法ですね。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def bubble_sort(n_list):
    start = time.time()
    swap_count = 0
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)
    flg = 1
    while flg:
        flg = 0
        for i in reversed(range(1, len(sorted_list))):
            if sorted_list[i] < sorted_list[i - 1]:
                swap_count += 1
                tmp = sorted_list[i]
                sorted_list[i] = sorted_list[i - 1]
                sorted_list[i - 1] = tmp
                flg = 1

    print(swap_count, swap_count)
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list
```

実装はかなり簡単です。reversed()で range()から発生させる数値を逆順にしています。
n_list[i]と n_list[i - 1]の大小を比較しているところの不等号を逆転させると昇順ソートと降順ソートを切り替えられます。

## テストと実行速度測定

### テストケース 1.ランダム状態からのソート

以下を実行。ランダムに１万個の整数リストを生成し、自作ソートメソッド適用後と python 組み込みのソートメソッド適用後で同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
orig_sort =bubble_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-40.png)組み込みソートと同様の結果を得られたが、実行にかなりの時間がかかってしまった。交換回数は約 2507 万回。

### テストケース 2.降順状態からのソート|計算量 O(n2)

ケース 1 と同様の手法でリストを作成し、.sort(reverse=True)で降順にソートをかけた。それに対し自作ソートメソッドと python 組み込みのソートメソッドを適用し、同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort(reverse=True)
orig_sort =bubble_sort(n_list)
py_sort = sorted(n_list)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-41.png)実行時間は 25 秒。交換回数は約 4999 万回で、最も交換回数とループを回す回数が多い場合はやはり時間がかかる。

### テストケース 3.昇順状態からのソート|計算量 O(n)

ケース 1 と同様の手法でリストを作成し、.sort()で昇順にソートをかけた。それに対し自作ソートメソッドと python 組み込みのソートメソッドを適用し、同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort()
orig_sort =bubble_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-42.png)挿入ソートと同じく、元々ソートされている状態からだと一回リストを走査するだけなのでかなり早い。
