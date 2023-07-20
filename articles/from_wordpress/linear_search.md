---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

> n 個の整数を含む数列 S と、q 個の異なる整数を含む数列 T を読み込み、T に含まれる整数の中で S に含まれるものの個数 C 出力するプログラムを作成してください。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_A](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/4/ALDS1_4_A)

## 解法

### 1.for ループを用いる方法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def linear_search(x, n_list):
    for n in n_list:
        if x == n:
            return True
    return False

S = [random.randint(1, 10000) for i in range(1000000)]
T = set([random.randint(1, 10000) for i in range(100)])

# 普通の線形探索
count = 0
start = time.time()
for t in T:
    if linear_search(t, S):
        count += 1
print(elapsed_time:{0}.format(time.time() - start))
print(count)
```

ただ for 文を回して 1 要素ずつ比較を行う方法。
上記コードの実行時間は約**0.03 秒**であった。

### 2. while 文と番兵を用いる方法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def linear_search_banpei(x, n_list):
    n_list.append(x)
    i = 0
    while x != n_list[i]:
        i += 1
    if i + 1 == len(n_list):
        n_list.pop(-1)
        return False
    n_list.pop(-1)
    return True

S = [random.randint(1, 10000) for i in range(1000000)]
T = set([random.randint(1, 10000) for i in range(100)])

# 番兵を用いた線形探索
count = 0
start = time.time()
for t in T:
    if linear_search_banpei(t, S):
        count += 1
print(elapsed_time:{0}.format(time.time() - start))
print(count)
```

「プログラミングコンテスト攻略のためのアルゴリズムとデータ構造」に載っていた、「番兵」となる要素をリストの最後に追加することで必ず while 文が終わるようにし、ループ中で行う要素比較の回数を少なくすることで効率化を図る実装方法。

1 の for ループを用いる方法だとループの継続判定と要素の数値が同じかの 2 つの判定をしなければならないが、2 の番兵を使う方法だとループ中で行うのがループの継続判定のみでいいため、効率的になると書いてあった。

だが実際に上記コードを動かすと、2 の番兵を使った方法は**0.08 秒**の実行時間がかかり、1 の単純な for ループを使った方法よりも実行に時間がかかってしまった。Python のリストは参照渡しであるため、2 の番兵を使った方法ではいちいちリストに要素を追加したり削除したりしているのだが、恐らくここで処理に時間がかかっており、理論とは逆の結果となっているのだろう。
