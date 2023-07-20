---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

> 名前 _name\*\*i_ と必要な処理時間 _time\*\*i_ を持つ _n_ 個のプロセスが順番に一列に並んでいます。ラウンドロビンスケジューリングと呼ばれる処理方法では、CPU がプロセスを順番に処理します。各プロセスは最大 _q_ ms（これをクオンタムと呼びます）だけ処理が実行されます。_q_ ms だけ処理を行っても、まだそのプロセスが完了しなければ、そのプロセスは列の最後尾に移動し、CPU は次のプロセスに割り当てられます。
>
> [https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/3/ALDS1_3_B](https://onlinejudge.u-aizu.ac.jp/courses/lesson/1/ALDS1/3/ALDS1_3_B)

例えば、クオンタムを 100 ms とし、次のようなプロセスキューを考えます。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">A(150) - B(80) - C(200) - D(200)
```

まずプロセス A が 100 ms だけ処理され残りの必要時間 50 ms を保持しキューの末尾に移動します。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">B(80) - C(200) - D(200) - A(50)
```

次にプロセス B が 80 ms だけ処理され、時刻 180 ms で終了し、キューから削除されます。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">C(200) - D(200) - A(50)
```

次にプロセス C が 100 ms だけ処理され、残りの必要時間 100 ms を保持し列の末尾に移動します。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">D(200) - A(50) - C(100)
```

このように、全てのプロセスが終了するまで処理を繰り返します。
ラウンドロビンスケジューリングをシミュレートするプログラムを作成してください。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import sys


class Queue:
    def __init__(self):
        self.Q = []

    def isEmpty(self):
        return len(self.Q) == 0

    def enqueue(self, x):
        self.Q.append(x)

    def dequeue(self):
        if self.isEmpty():
            sys.exit(Empty)
        else:
            return self.Q.pop(0)


# ローカルチェック用
# process_list = [('p1',150),('p2',80),('p3',200),('p4',350),('p5',20)]
# power= 100

n, power = map(int, raw_input().split( ))
process_list = [map(str, raw_input().split( )) for i in range(n)]
process_list = [(p[0], int(p[1])) for p in process_list]

q = Queue()
for p in process_list:
    q.enqueue(p)
elapsed_time = 0
while not q.isEmpty():
    process = q.dequeue()
    if process[1] > power:
        elapsed_time += power
        q.enqueue((process[0], process[1] - power))
    else:
        elapsed_time += process[1]
        print(str(process[0]) +   + str(elapsed_time))
```

キュー構造を作り、その中でデータを出し入れしつつ、処理が完了したらプロセス ID と経過時間を print するコードです。単純ですね。
