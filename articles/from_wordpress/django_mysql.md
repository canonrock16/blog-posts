---
title: ""
emoji: "🤖"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

<!-- wp:heading -->## はじめに

Django で MySQL の JSON 型を使いたい時に必要な作業を書きました。

#### 前提条件

- MySQL が既に導入・DB 構築されており、*mysql.server start*で起動されている状態である
- mysqlclient を導入し、MySQL に接続するための settings.py の設定が済んでいる(今回使用する Django-mysql は[mysqlclient を前提としている](https://django-mysql.readthedocs.io/en/latest/installation.html))

#### 今回使用したモジュールのバージョン

- Django v2.1.15
- mysqlclient v1.4.6
- Django-mysql v3.3.0

## 作業手順

1. Django-mysql のインストール
2. モデル追加
3. settings.py の編集

### 1.Django-mysql のインストール

Django で MySQL 用の JSONfield を使えるようにしてくれるモジュール、[Django-mysql](https://django-mysql.readthedocs.io/en/latest/index.html)をインストールします。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">pip install django-mysql
```

### 2.モデル追加

自分のモデル定義に JSONfield を追加。model の継承元も django_mysql のものにする。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">from django_mysql.models import JSONField, Model

class ShopItem(Model):
    name = models.CharField(max_length=200)
    attrs = JSONField()

    def __str__(self):
        return self.name
```

### 3.settings.py の編集

settings.py の INSTALLED_APPS に以下を追加する。
(ちなみに、自分の環境では何故かこの設定を行わなくても JSON 型のカラムが生成されてしまったが、[公式ドキュメントにある通り](https://django-mysql.readthedocs.io/en/latest/installation.html)きちんと指定した方が無用なトラブルを避けられると思う。)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">INSTALLED_APPS = (
    ...
    'django_mysql',
)
```

公式ドキュメントによると以上で作業は完了なのだが、自分の環境では

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">?: (django_mysql.W003) The character set is not utf8mb4 for database connection 'default'<br>
    HINT: The default 'utf8' character set does not include support for all Unicode characters. It's strongly recommended you move to use 'utf8mb4'. See: https://django-mysql.readthedocs.io/en/latest/checks.html#django-mysql-w003-utf8mb4
```

というエラーが出てしまった。これを解決するため、settings.py の DB 接続情報に\'OPTIONS\'項目を追加し、utf8mb4 という文字コードを指定する。

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="python" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'USER': 'xxxxx',
        'PASSWORD': 'xxxxx',
        'HOST': 'localhost',
        'OPTIONS': {
            'charset': 'utf8mb4',  # <--- Use this
        }
    }
}
```

#### utf8mb4 とは何なのか

通常の utf-8 は[1~4 バイトで 1 文字を表します](https://ja.wikipedia.org/wiki/UTF-8)が、何故か MySQL において utf-8 と名付けられている charset は[1~3 バイトまでしか対応していない](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-utf8.html)ため、普段あまり使われないような漢字や絵文字が正しく登録されません。

この問題に対応できる MySQL での charset が utf8mb4 で、1~4 バイトまで対応しているため、基本的にはこちらを使うのが良いようです。

最初、自分の環境では特にデフォルトの charset を指定していなかったのですが、その場合は charset が utf8mb4 になり、今回のエラーに繋がったようです。

#### デフォルトの文字コードを指定する

やはり文字コードは明示的に指定した方が良いので、MySQL の設定ファイルに設定を追加します。自身の環境の my.cnf に以下を追記します。[引用](https://qiita.com/deco/items/bfa125ae45c16811536a)

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">[mysqld]
...
character-set-server=utf8mb4

[client]
default-character-set=utf8mb4
```

自身の環境の my.cnf の場所が不明な場合、

```
<pre class="EnlighterJSRAW" data-enlighter-group="" data-enlighter-highlight="" data-enlighter-language="generic" data-enlighter-linenumbers="" data-enlighter-lineoffset="" data-enlighter-theme="" data-enlighter-title="">mysql --help | grep my.cnf
```

で調べられます。[引用](https://qiita.com/is0me/items/12629e3602ebb27c26a4)

今回は以上です。
