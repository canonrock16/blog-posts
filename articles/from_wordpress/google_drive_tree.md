---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 動機

google drive 上にある議事録などのファイルを階層構造付で一覧化したい時があった。
ネットで検索すると google app script を使用して google スプレッドシートに吐き出すというやり方の記事が多いが、tree コマンドの結果のような形式でファイル一覧が作成したかったため、やってみた。

## 手順

### 1.ファイル一覧を作成したい google drive にログインし、Google Colaborarory のファイルを作る。

google drive→ 新規 → その他 →Google Colaborarory

### 2.以下の 4 セルの内容を貼り付ける

```python
# google　driveをGoogle Colaboraroryからアクセスできる場所にマウントする
from google.colab import drive
drive.mount('/content/drive')
```

```python
# Google Colaborarory環境にはtreeが入っていないので入れる
!sudo apt-get install tree
```

```python
# /path/to/dirはtreeを発行したいディレクトリに変更
%cd drive/MyDrive/path/to/dir
```

```python
!tree -N
```

### 3.セルを順番に実行していく

以下のセルの実行をすると、google drive への認証を求められる。指示に沿って認証を行う。

```python
from google.colab import drive
drive.mount('/content/drive')
```

順番に実行していくと以下のような結果が得られる。

```python
!tree -N

.
├── dir1
│   ├── 基本設計
│   │   ├── file1
│   │   ├── file2
│   │   ├── file3
│   │   ├── dir2
│   │   │   ├── file4
│   │   │   └── file5
│   │   ├── file6
.
.
.
```

## なぜ Google Colaborarory を使うのか

tree コマンドを発行するだけであれば google API を使用すればできそうではあるものの、会社で使用している google drive の場合は外部 API 認証が出来ない場合があるため、Google Colaborarory を使用した。

外部 API 認証が出来ない場合とは、google 関連のサービスに二段階認証を設定している場合。
二段階認証を ON にしていなければ、設定の［安全性の低いアプリの許可］から許可を行うことで外部 API から drive にアクセスできるようになるが、会社全体で二段階認証を ON にしている場合、tree コマンドを発行したいというだけで二段階認証を OFF にするわけにもいかない。そういう場合は Google Colaborarory から drive にアクセスすることで、認証で弾かれる事態を防ぐことができる。
[参考](https://support.justsystems.com/faq/1032/app/servlet/qadoc?QID=054933#:~:text=%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6%E3%81%A7Google%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9,%E7%84%A1%E5%8A%B9%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%97%E3%81%BE%E3%81%99%E3%80%82)

他の方法として、OAuth2.0 に対応させた形で google drive にアクセスできるようにすればそれでも tree コマンドは実行可能である。(が、tree を発行するためだけにそこまでやるのは面倒な気もする。)
[参考 2](https://support.google.com/a/answer/6260879?hl=ja#zippy=%2C%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92%E5%AE%89%E5%85%A8%E6%80%A7%E3%81%AE%E9%AB%98%E3%81%84%E3%82%82%E3%81%AE%E3%81%AB%E5%A4%89%E6%9B%B4%E3%81%99%E3%82%8B)
