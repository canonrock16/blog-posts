---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

[AIZU ONLINE JUDGE を参照](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_5_C&lang=ja)

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">from math import cos, sin, radians


def koch(n, a, b):
    if n == 0:
        return
    s = [0, 0]
    t = [0, 0]
    u = [0, 0]

    s[0] = (2 * a[0] + 1 * b[0]) / 3
    s[1] = (2 * a[1] + 1 * b[1]) / 3
    t[0] = (1 * a[0] + 2 * b[0]) / 3
    t[1] = (1 * a[1] + 2 * b[1]) / 3
    u[0] = (t[0] - s[0]) * cos(radians(60)) - (t[1] - s[1]) * sin(radians(60)) + s[0]
    u[1] = (t[0] - s[0]) * sin(radians(60)) - (t[1] - s[1]) * cos(radians(60)) + s[1]

    koch(n - 1, a, s)
    print(s[0], s[1])
    koch(n - 1, s, u)
    print(u[0], s[1])
    koch(n - 1, u, t)
    print(t[0], t[1])
    koch(n - 1, t, b)


a = [0, 0]
b = [100, 0]
n = 1

print(a[0], a[1])
koch(n, a, b)
print(b[0], b[1])
```

解説は TBD.
