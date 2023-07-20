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

> 挿入ソート（Insertion Sort）は、手持ちのトランプを並び替えるときに使われる、自然で思い付きやすいアルゴリズムの１つです。片手に持ったトランプを左から小さい順に並べる場合、１枚ずつカードを取り出して、それをその時点ですでにソートされている並びの適切な位置に挿入していくことによって、カードを並べ替えることができます。
> 挿入ソートは次のようなアルゴリズムになります。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/1/ALDS1_1_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/1/ALDS1_1_A)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">insertionSort(A, N) // N個の要素を含む0-オリジンの配列A
   for i が 1 から N-1 まで
     v = A[i]
     j = i - 1
     while j >= 0 かつ A[j] > v
       A[j+1] = A[j]
       j--
     A[j+1] = v
```

> _N_ 個の要素を含む数列 _A_ を昇順に並び替える挿入ソートのプログラムを作成してください。上の疑似コードに従いアルゴリズムを実装してください。アルゴリズムの動作を確認するため、各計算ステップでの配列（入力直後の並びと、各 _i_ の処理が終了した直後の並び）を出力してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/1/ALDS1_1_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/1/ALDS1_1_A)

ソートアルゴリズムの一つ、挿入ソートの実装です。左から順番にポインタを動かし、ポインタがある箇所よりも左では常にソートされた状態に保つようなソート手法ですね。ソート手法自体の詳細な説明はわかりやすいものが既に色々な書籍やサイトに記載されていると思うので、ここでは省きます。

ソートされる対象の数値の並び順によって計算量が大きく変わるソート手法で、昇順にソートしたい時に元々の配列が降順になっている場合は O(n2)の計算量になってしまいます。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def insertion_sort(n_list):
    start = time.time()
    # Pythonは参照渡しであるので、新規オブジェクトを作成しないと元のリストが変更される
    sorted_list = list(n_list)

    # リストの1番目からまず親ポインタを動かす
    for i in range(1, len(sorted_list)):
        # 今親ポインタがある要素を取得
        pointed_value = sorted_list[i]
        # 子ポインタ(親ポインタがある場所から左端に向かって動いていくポインタ)のインデックスを親ポインタの左隣で初期化
        j = i - 1
        # 子ポインタのある場所の要素と親ポインタのある場所の要素を比較し、子ポインタのある場所の要素の方が大きかったら、子ポインタの右隣を子ポインタの要素に置き換える
        # 子ポインタのある場所の要素が親ポインタのある場所の要素よりも小さくなるか、小ポインタのインデックスが-1になるまでループを回す。
        while j >= 0 and (sorted_list[j] > pointed_value):
            sorted_list[j + 1] = sorted_list[j]
            # Pythonでは--のような演算子は使えない
            j -= 1
        # 子ポインタが停止した要素(親ポインタがある要素よりも小さい要素か、インデックスが-1の場所)の右隣に親ポインタがある要素を入れる
        sorted_list[j + 1] = pointed_value
    print(elapsed_time:{0}.format(time.time() - start))
    return sorted_list
```

(コメント書きすぎで見づらくなりました。)
n_list[j]と pointed_value の大小関係の比較を逆にすると降順ソートと昇順ソートを切り替えられます。

## テストと実行速度測定

### テストケース 1.ランダム状態からのソート

以下を実行。ランダムに１万個の整数を生成し、python 組み込みのソートメソッドと同様のリストが得られるかをテストした。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
orig_sort = insertion_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-43.png)実行時間は 7 秒くらい。ソート結果は組み込みと等しかった。

### テストケース 2.降順状態からのソート|計算量 O(n2)

理論的には最も時間がかかるはずの、降順状態からの昇順へのソート。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort(reverse=True)
orig_sort = insertion_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-44.png)実行時間は 13 秒くらい。

### テストケース 3.昇順状態からのソート|計算量 O(n)

理論的には最も時間がかからないはず。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">n_list = [random.randint(1, 10 ** 9) for i in range(10000)]
n_list.sort()
orig_sort = insertion_sort(n_list)
py_sort = sorted(n_list)
print(orig_sort == py_sort)
```

#### 結果

![](https://lonely-journalclub.com/wp-content/uploads/2019/12/image-45.png)実行時間は 0.006 秒。想定通り一番早かった。

## まとめ

いつものことだが、O(n2)と O(n)の実行速度の違いが圧倒的。挿入ソートは既にある程度整列されている状態からの使用が向いていそう。
