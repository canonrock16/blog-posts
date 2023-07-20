---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

以下の命令を受けつける双方向連結リストを実装してください。

- insert x: 連結リストの先頭にキー _x_ を持つ要素を継ぎ足す。
- delete x: キー _x_ を持つ最初の要素を連結リストから削除する。そのような要素が存在しない場合は何もしない。
- deleteFirst: リストの先頭の要素を削除する。
- deleteLast: リストの末尾の要素を削除する。

## 解法

※以下実装は効率が悪く、AIZU ONLINE JUDGE の命令数が多い No.9,No10 のテストは時間切れで失敗します。効率化の方法がわかったら更新します。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">import sys


class Cell:
    def __init__(self, x):
        self.x = x
        self.next = None
        self.prev = None


class Doubly_linked_list:
    def __init__(self):
        self.head = None

    def insert_tail(self, x):
        new = Cell(x)
        # まだ一つも要素が無い場合
        if self.head is None:
            self.head = new
            new.prev = new
            new.next = new

        # headの一つ前は必ず最後の要素になるので、最後の要素のnextに新しいcellを指定する
        self.head.prev.next = new
        # 新しいcellのprebにheadの一つ前の最後の要素を指定
        new.prev = self.head.prev
        # 最後に追加した新しいcellのnextはheadにする
        new.next = self.head
        # headのprebを新しい要素に指定
        self.head.prev = new

    def insert_head(self, x):
        new = Cell(x)
        # まだ一つも要素が無い場合
        if self.head is None:
            self.head = new
            new.prev = new
            new.next = new

        new.next = self.head
        new.prev = self.head.prev
        self.head.prev.next = new
        self.head.prev = new
        self.head = new

    def delete(self, x):
        tmp_cell = self.head
        head_cell = self.head
        # 1つも要素が無い場合
        if self.head is None:
            sys.exit(List is empty)

        while tmp_cell:
            if tmp_cell.x == x:
                tmp_cell.prev.next = tmp_cell.next
                tmp_cell.next.prev = tmp_cell.prev
                if tmp_cell == self.head:
                    self.head = self.head.next
                return
            tmp_cell = tmp_cell.next
            if tmp_cell == head_cell:
                # sys.exit(delete value not in list)
                return

    def deleteFirst(self):
        # 1つも要素が無い場合
        if self.head is None:
            sys.exit(List is empty)
        # 要素が一つしかない場合
        if self.head.next == self.head:
            self.head = None
            return

        self.head.prev.next = self.head.next
        self.head.next.prev = self.head.prev
        self.head = self.head.next

    def deleteLast(self):
        # 1つも要素が無い場合
        if self.head is None:
            sys.exit(List is empty)

        # 要素が一つしかない場合
        if self.head.next == self.head:
            self.head = None
            return

        self.head.prev.prev.next = self.head
        self.head.prev = self.head.prev.prev

    def show(self):
        tmp_cell = self.head
        while tmp_cell:
            print(tmp_cell.x, prev, tmp_cell.prev.x, next, tmp_cell.next.x)
            tmp_cell = tmp_cell.next
            if tmp_cell == self.head:
                return

    def answer(self):
        answer = []
        tmp_cell = self.head
        while tmp_cell:
            answer.append(tmp_cell.x)
            tmp_cell = tmp_cell.next
            if tmp_cell == self.head:
                return answer


d = Doubly_linked_list()
n = int(raw_input())
for i in range(n):
    p = raw_input().split()
    if p[0] == insert:
        d.insert_head(p[1])
    elif p[0] == delete:
        d.delete(p[1])
    elif p[0] == deleteFirst:
        d.deleteFirst()
    elif p[0] == deleteLast:
        d.deleteLast()
print( .join(map(str, d.answer())))


# ローカルテスト
d = Doubly_linked_list()
d.insert_head(1)
d.insert_head(2)
d.insert_head(3)
d.insert_head(4)
d.insert_head(5)
d.insert_head(6)
d.insert_head(7)
d.insert_head(8)
d.delete(5)
d.delete(7)
d.insert_head(9)
d.deleteLast()
d.deleteLast()
d.insert_head(10)
d.deleteFirst()
d.deleteFirst()
d.delete(8)
d.insert_head(7)
d.insert_head(8)
d.delete(4)
d.insert_head(1)
# print(----show----)
# d.show()
print( .join(map(str, d.answer())))
```

それぞれの cell の prev,next を書き換えることで insert や delete といった機能を実現しています。

Python 標準のリストを使えばもっと早く問題自体を解くことは出来るのでしょうか、この章の趣旨は双方向連結リストを自分で実装することなので、動作は遅いですがこちらで回答ということにしました。
