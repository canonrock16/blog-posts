---
title: "pythonの依存関係管理ツールpoetryの基本的な使い方と注意点"
emoji: "🐍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","poetry","pip","パッケージ管理"]
published: false
---

# はじめに
2020/10時点でのpoetryのごく基本的な使い方を記載します。poetryと他依存関係ツールの違いや、より詳細な使い方は他の方が書いた記事や公式ドキュメントがあるのでそちらを参照ください。

#### 動作確認環境
OS:macOS Catalina v10.15.4
poetry:v1.0.8

# poetryのインストール
### インストール
```shell
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
```
### 初期設定
以下のコマンドを打つと、poetryが作成する仮想環境(.venvフォルダ)がpoetryを使ってパッケージ管理を行うプロジェクトの直下に作成されるようになります。

```shell
poetry config virtualenvs.in-project true
```
これが何故必要かというと、vscodeを使用する際、この設定をしていない場合はいちいち仮想環境のpathをvscodeのsettings.jsonに書き加えるなど仮想環境をvscodeに認識させるための手順が必要になってしまいますが、最初にこの設定をしておけばその手間を省くことができます。

# プロジェクトへのpoetry導入
### 導入
以下のコマンドを打つとパッケージ一覧を管理する`pyproject.toml` が直下に作成されます。

```shell
poetry init
```
対話式でいくつかプロジェクトの名称やauthor等について質問されますので、適宜答えてください。とりあえずエンター連打でもまぁ良いです。
プロジェクトを開始する際、`poetry new`としても良いのですが、こちらのコマンドではディレクトリやREADMEまで作ってくれてしまいます。ディレクトリ構造等は自前で作りたい場合が多いと思いますので、`poetry init`で良いと思います。

### パッケージの追加
`poetry init`で作成された`pyproject.toml`に使用するライブラリの情報を書き込みつつ、仮想環境へのインストールを行います。

```shell
# とりあえず最新のものがあれば良い場合
poetry add <パッケージ名>

# バージョン指定も可能
poetry add "pendulum>=2.0.5"

# gitからも取ってこれる
poetry add git+https://github.com/sdispater/pendulum.git
```

### パッケージの削除
`pyproject.toml`からライブラリを削除し、仮想環境からも削除します。

```shell
poetry remove <パッケージ名>
```
### 仮想環境を使ってコマンドを実行

```shell
poetry run <コマンド>
# 例えば、poetryを使って入れたパッケージを有効化した状態で、poetryで入れたjupyterlabを立ち上げたい時
poetry run jupyter lab
```


### パッケージのupdate
```shell
# 一部のパッケージのみアップデートする場合
poetry update <パッケージ名>

# 全てのパッケージをアップデートする場合
poetry update
```

### `pyproject.toml`記載のパッケージをインストールする場合
他人が作った`pyproject.toml`から一括でパッケージをインストールしたい場合などに使用

```shell
poetry install
```

### poetry自体のアップデート
※検索するとたまに出てくる、`poetry self:update`はv1.0以前のpoetryのコマンドのようなので、現在は以下コマンドを使用してください。

```shell
poetry self update
```


# 注意点
### パッケージ名の指定
`poetry init`時に指定するパッケージ名を、`poetry add`でインストールしたいパッケージ名と同一にすると以下のエラーが出る場合があります。

```pyproject.toml
[tool.poetry]
name = "kedro"
```

```shell
❯ poetry add kedro
Using version ^0.16.1 for kedro

Updating dependencies
Resolving dependencies... (0.0s)

[AssertionError]
```
この場合、`name = "kedro_practice"`など、別のものに変えてあげれば正常にインストールできるようになります。
参考issue:https://github.com/python-poetry/poetry/issues/1295


### pythonバージョンの指定
`poetry init`を行う際に特別設定をしなければ、poetryが想定するpythonのバージョンについて`pyproject.toml`に以下のように記載されます。

```toml
[tool.poetry.dependencies]
python = "^3.8"
```
^3.8が何を意味するかと言うと、**3.8<=pythonのバージョン<4.0**です。左端のメジャーバージョンを表す数値が変わらない最大のバージョンまで想定するというバージョン指定になります。(詳細な情報は[公式ドキュメント](https://python-poetry.org/docs/versions/)を見てください。)
`poetry add`等でパッケージのインストールを行う際、poetryは導入パッケージが`pyproject.toml`で設定されているpythonのバージョンで使用できるかどうか判定を行いますが、**その判定対象のpythonのバージョンは想定される全てのバージョンになります。**つまり、上記の例だと、「3.8<=pythonのバージョン<4.0」を満たす全てのバージョン、つまり**v3.8とv3.9が判定対象になります。**
例えばkedroというパッケージは2020/06/07現在で対応するpythonのバージョンが`3.6`,`3.7`,`3.8`で、`3.9`には対応していません。

`pyproject.toml`の`[tool.poetry.dependencies]`欄に`python = "^3.8"`と記載した状態でこういったパッケージを`poetry add`しようとすると、以下のエラーが発生します。

```shell-session
[SolverProblemError]
The current project's Python requirement (^3.8) is not compatible with some of the required packages Python requirement:
  - kedro requires Python >=3.6, <3.9

Because no versions of kedro match >0.16.1,<0.17.0
 and kedro (0.16.1) requires Python >=3.6, <3.9, kedro is forbidden.
So, because kedro-test depends on kedro (^0.16.1), version solving failed.
```

バージョン3.8にしてるじゃん！なんでダメなの？！と時間をつぶす羽目になります。

解決策としては、v3.8のみを想定するpythonのバージョンとするよう、以下のように`pyproject.toml`を記載する必要があります。

```toml
[tool.poetry.dependencies]
python = "^3.8,<3.9"
#もしくは
python = "=3.8"
```
より詳細な情報が欲しい方は、poetryのgithubのissueを追ってみてください。
https://github.com/python-poetry/poetry/issues/1413
https://github.com/python-poetry/poetry/issues/2444

### パッケージのupdate
上にパッケージのupdateは`poetry update <パッケージ名>`でできると書きましたが、
**このコマンドでupdateできるのは`pyproject.toml`に記載されているバージョン指定の範囲内まで**となります。
`poetry add numpy`でパッケージを導入した場合、`pyproject.toml`には`numpy = "^1.18.2"`と記載され、「**1.18.2<=numpyのバージョン<2.0**」が指定されます。(詳細な情報は[公式ドキュメント](https://python-poetry.org/docs/versions/)を見てください。)

このため、メジャーバージョンのアップデートを行いたい場合、
- `pyproject.toml`を直接編集してから`poetry update <パッケージ名>`をする
か、面倒な場合は
- `poetry add <パッケージ名>@latest`
を行う必要があります。





# さいごに
本記事では触れていませんが、pyenvと組み合わせて使うといい感じです。pipよりだいぶ快適なので、ぜひ使ってみてください。
