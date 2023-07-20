---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:heading -->## はじめに

python コードを書いている時、こんな状態に遭遇した。

**パターン 1**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def aaa():
    print(a)

a = 1
aaa()
#実行でき、1が標準出力される。
```

**パターン 2**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def aaa():
    a = a + 1

a = 1
aaa()
#UnboundLocalError: local variable 'a' referenced before assignment
```

関数の中で外部の変数を参照することは出来るのに、代入することは出来ないのである。

だが、変数の型をリストにすると、

**パターン 3**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def bbb():
    print(b)

b = [1,2,3]
bbb()
#実行でき、[1,2,3]が標準出力される。
```

**パターン 4**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def bbb():
    b[0] = 10
    print(b)

b = [1,2,3]
bbb()
#実行でき、[10,2,3]が標準出力される。
```

外部の変数を参照することも代入することもできる。

Python では関数内からグローバル変数を扱うときは global 宣言、関数内関数から親関数の変数を扱うときは nonlocal 宣言をする必要があり、これらの宣言をすればパターン 2 のような局面でもエラー無く実行できる。

だが、パターン 1~4 の結果を見比べて、以下のような疑問が生まれた。

1. **パターン 1 において、global 宣言をしていないのになぜ print は出来たのか？**
2. **パターン 3,4 において、global 宣言をしていないのになぜ print も代入も出来たのか？**

本記事ではこれらの疑問に対する答えを調査し、書いていこうと思う。

## 1.パターン 1 において、global 宣言をしていないのになぜ print は出来たのか？

Python ドキュメントで関連する箇所を見てみた。

> 関数を 実行 (execution) するとき、関数のローカル変数のために使われる新たなシンボルテーブル (symbol table) が用意されます。 もっと正確にいうと、**関数内で変数への代入を行うと、その値はすべてこのローカルなシンボルテーブルに記憶されます。** 一方、**変数の参照を行うと、まずローカルなシンボルテーブルが検索され、次にさらに外側の関数のローカルなシンボルテーブルを検索し、その後グローバルなシンボルテーブルを調べ、最後に組み込みの名前テーブルを調べます。** 従って、関数の中では (グローバル変数が global 文で指定されていたり、外側の関数の変数が nonlocal 文で指定されていない限り) グローバル変数や外側の関数の変数に直接値を代入できませんが、参照することはできます。
>
> [https://docs.python.org/ja/3/tutorial/controlflow.html?highlight=nonlocal](https://docs.python.org/ja/3/tutorial/controlflow.html?highlight=nonlocal)

上記部分によると、

**変数の参照時はローカルのシンボルテーブルから組み込みの名前テーブルまで遡って検索してくれるが、**

**変数の代入時はローカルのシンボルテーブルまでしか検索しないため、そんな変数は存在しないと言われる**

ようであった。

## 2.パターン 3,4 において、global 宣言をしていないのになぜ print も代入も出来たのか？

これをはっきりさせるため、追加で実験を行った。

**パターン 5**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def bbb():
    b[0] = 10
    print(locals, locals())
    print(globals, globals())

b = [1, 2, 3]
bbb()
print(b)
#locals {}
#globals {'__name__': '__main__', '__doc__': None, '__package__': None, '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': None, '__annotations__': {}, '__builtins__': <module 'builtins' (built-in)>, 'bbb': <function bbb at 0x104e156a8>, 'b': [10, 2, 3]}
#[10, 2, 3]
```

locals(),globals()でローカル変数リスト、global 変数リストを確認している。

list 型だからといってローカル変数に自動で組み込まれる、などということは無いことが確認できた。

更にもう一つ実行してみた。

**パターン 6**

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def bbb():
    b = b.append(10)
    print(b)
    print(locals, locals())
    print(globals, globals())

b = [1, 2, 3]
bbb()
#UnboundLocalError: local variable 'b' referenced before assignment
```

これらの結果から、

**パターン 5 では b[0]と要素の指定をしている際に変数の参照を行っており、自動でローカルのシンボルテーブルから組み込みの名前テーブルまで遡って検索されたため、その後の代入も行うことができた。
パターン 6 では b.append(10)と変数の参照を行っていないためローカルのシンボルテーブルのみ検索され、ローカルのシンボルテーブルには変数 b が存在しなかったため、エラーとなった。**

のではないかと思う。
