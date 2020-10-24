---
title: "pandasのto_sqlメソッドでのinsertが遅い時の対処法"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pandas"]
published: false
---

## 先に結論
以下の様に、to_sqlメソッドのmethod引数に'multi'を指定すれば良い。
'multi'を指定しなかった際に30分近くかかっていた処理が10秒程度で終わるようになった。
```python
df.to_sql(...,method='multi')
```

## なぜこうなるのか
to_sqlメソッドの`method`引数に何も指定せずに実行した場合、Dataframe内のデータに対して1行ずつinsert文を生成・実行するため。
`method='multi'`を指定してやると複数行を一括でinsertする文を生成・実行するため、DBへ接続するための時間がかからなくなった模様。
insertしたい行数が多く、DB側の一度にinsertできる行数の上限を超えそうな場合は`chunksize`引数に上限値を設定してやれば良い。



