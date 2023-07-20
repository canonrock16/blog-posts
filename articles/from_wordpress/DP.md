---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:block {ref:510} /--><!-- wp:heading -->## 問題

[AIZU ONLINE JUDGE のこの問題](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_5_A&lang=ja)を動的計画法を使用して解きます。

## 解法

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">def solve(i, target):
    global input_list
    global dp
    if str(i) + _ + str(target) in dp.keys():
        return dp[str(i) + _ + str(target)]
    if target == 0:
        dp[str(i) + _ + str(target)] = True
    elif i >= len(input_list):
        dp[str(i) + _ + str(target)] = False
    elif solve(i + 1, target):
        dp[str(i) + _ + str(target)] = True
    elif solve(i + 1, target - input_list[i]):
        dp[str(i) + _ + str(target)] = True
    else:
        dp[str(i) + _ + str(target)] = False

    return dp[str(i) + _ + str(target)]


target = 5
input_list = [1, 5, 7, 10, 21]
dp = {}
solve(0, target)

target = 4
input_list = [2, 4, 17, 8]
dp = {}
solve(0, target)

target = 25
input_list = [2, 4, 17, 8]
dp = {}
solve(0, target)
```

既に計算した組み合わせの結果を辞書に格納しておき、再度の計算の手間を省くことで実行時間を削減できる。実際にそのようになるのか、動的計画法を使用していない ver と使用した ver で実行時間を比較してみた。

以下のコードを実行して実行時間を計測した。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">target = 79901
input_list = [1577, 8512, 7435, 2613, 4710, 6134, 6097, 6533, 8106, 3367, 3407, 429, 7974, 3162, 4639, 6563, 2378, 7852, 617, 9297, 4642, 9315, 2314, 6831, 1936, 2525, 8904, 7477, 3149, 3879, 2948, 7634, 3363, 7147, 9037, 4721, 1610, 8136, 2724, 3532, 6428, 5434, 6815, 7311, 3671, 6990, 8523, 1190, 9202, 2541, 1590, 7986, 2507, 2476, 2780, 3276, 49, 153, 8008, 9430, 2525, 7196, 6625, 1329, 4876, 7981, 6637, 9797, 9233, 9674, 7545, 8743, 6087, 8041, 9797, 8988, 246, 2340, 5096, 5255, 4289, 986, 7492, 8794, 6492, 6423, 5793, 8470, 5904, 9564, 5304, 2748, 8603, 5812, 5512, 2923, 175, 3718, 931, 5614]
start = time.time()
solve(0, target)
print(elapsed_time:{0}.format(time.time() - start))
```

#### 動的計画法不使用 ver

使用したコードは[この投稿](https://lonely-journalclub.com/algorithm/mynavi-book/post-705/)で作成したもの。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">elapsed_time:0.4728531837463379
```

#### 動的計画法使用 ver

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">elapsed_time:0.0010328292846679688
```

確かに動的計画法を使用した方が実行速度が早かった。
