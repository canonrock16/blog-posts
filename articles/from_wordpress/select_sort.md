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

> 選択ソートは、各計算ステップで１つの最小値を「選択」していく、直観的なアルゴリズムです。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_B](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_B)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">selectionSort(A, N) // N個の要素を含む0-オリジンの配列A
   for i が 0 から N-1 まで
     minj = i
     for j が i から N-1 まで
       if A[j] < A[minj]
         minj = j
     A[i] と A[minj] を交換
```

> 数列 A を読み込み、選択ソートのアルゴリズムで昇順に並び替え出力するプログラムを作成してください。上の疑似コードに従いアルゴリズムを実装してください。
> 疑似コード 7 行目で、_i_ と _minj_ が異なり実際に交換が行われた回数も出力してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_B](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/all/ALDS1_2_B)

ソートアルゴリズムの一つ、選択ソートの問題です。左端からポインタを動かしていって、ポインタよりも右にある数の中で一番小さい数とポインタがあたっている数を入れ替えていくことでソートを行います。
数の並び順が元から昇順にソートされていても、左端から右端へのポインタ移動、ポインタよりも右側の領域での最小値探索は必ず行われるので、計算量は O(n2)になります。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def selection_sort(n_list):
    start = time.time()
    swap_count = 0
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)

    for i in range(len(sorted_list)):
        min_j = i
        for j in range(i, len(sorted_list)):
            if sorted_list[j] < sorted_list[min_j]:
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

min_j の初期値を左端から右端に動くポインタのインデックスにしておき、現在の min_j の要素よりも小さい要素が見つかったら両者を入れ替えるという動作を繰り返しています。現在ポインタがあたっている要素よりも小さい要素が見つからなかった場合、自分自身との入れ替えを実行する実装になっています。
n_list[j] < n_list[min_j]の箇所の不等号を逆転させることで昇順と降順を切り替えられます。

## テストと実行速度測定

### テストケース 1.ランダム状態からのソート | 計算量 O(n2)

以下を実行。ランダムに１万個の整数リストを生成し、自作ソートメソッド適用後と python 組み込みのソートメソッド適用後で同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
orig_sort = selection_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-36.png)組み込みソートと同様の結果が得られた。交換回数は約１万回で、バブルソートと比較して少なく抑えられている。

### テストケース 2.降順状態からのソート | 計算量 O(n2)

ケース 1 と同様の手法でリストを作成し、.sort(reverse=True)で降順にソートをかけた。それに対し自作ソートメソッドと python 組み込みのソートメソッドを適用し、同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort(reverse=True)
orig_sort = selection_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-37.png)5000 回と少ない交換回数で抑えられた。バブルソートの約 25 秒の実行時間と比較し、選択ソートの実行時間は 6 秒であり、かなり早いソート手法であることがわかる。

### テストケース 3.昇順状態からのソート | 計算量 O(n2)

ケース 1 と同様の手法でリストを作成し、.sort()で昇順にソートをかけた。それに対し自作ソートメソッドと python 組み込みのソートメソッドを適用し、同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort()
orig_sort = selection_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-38.png)交換回数は 0 回だが降順状態からのソートとほぼ同程度の時間がかかっている。元々昇順にソートされていたとしても計算量は O(n2)なのであまり処理時間がかわらないのは仕方がないことではあるが、もう少し早くならないだろうか。

今の実装だとポインタがあたっている要素よりも小さい要素が無くても自分自身との交換をしているので、ここを効率化したらちょっとは早くなるかも。
そう思って自分自身との交換をしない ver での実験もしてみた。コードと結果は以下。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def selection_sort2(n_list):
    start = time.time()
    swap_count = 0
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)

    for i in range(len(sorted_list)):
        min_j = i
        for j in range(i, len(sorted_list)):
            if sorted_list[j] < sorted_list[min_j]:
                min_j = j
        if i != min_j:
            swap_count += 1
            # n_list[i]よりも小さい要素があった場合のみ交換処理を行う
            tmp = sorted_list[i]
            sorted_list[i] = sorted_list[min_j]
            sorted_list[min_j] = tmp

    print(swap_count, swap_count)
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-39.png)処理時間は変わらず 5 秒程度だった。要素の交換処理にかかる時間は取るに足らないもので、やはり処理時間の大半はループ処理と数値の比較処理に費やされている模様。

## まとめ

ランダム順からのソート、降順からのソートではバブルソートよりも実行時間が早いが、昇順からのソートではバブルソートの方が圧倒的に早い。使い所を考えたほうが良さそうだ。
(もっとも、今は組み込みのソートが既に十分最適化されているのでとりあえずそれを使っておけば問題無さそうではあるのだが・・・。)
