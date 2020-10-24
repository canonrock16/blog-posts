---
title: "機械学習パイプライン構築ツールkedroの基本的な使い方"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","machine learning","kedro","pipeline"]
published: false
---

# はじめに
### kedroとは
機械学習のデータ読み込み〜前処理〜モデル作成〜精度検証までの一連の流れを、再利用可能なモジュールを組み合わせたパイプラインとして実行するためのフレームワークです。
この記事では、kedroの基本的な使い方を、機械学習界隈ではおなじみの[Irisデータセット](https://archive.ics.uci.edu/ml/datasets/iris)へのロジスティック回帰を行う処理を通して見ていきます。

### 注意
インストールやデバッグ実行に関してはpythonの依存関係管理ツールpoetryを使用した場合の手法を記載しています。pipやconda,pipenvを使った場合のやり方は公式ドキュメントにわかりやすく書いてあるので、コマンドはそちらをご覧ください。
poetry自体のインストールと使い方は簡単にまとめた[記事](https://qiita.com/canonrock16/items/f77ee2a2df9be5b8cc37)があるので、こちらをご覧ください。

# kedroのインストール

```shell
# kedroをインストール。もしエラーが出たら「はじめに」で紹介したpoetryの記事の「注意事項」欄を見てみてください
poetry add kedro

# インストールを確認.ローマ字でkedroと書かれたアスキーアートが出てくればOKです。
poetry run kedro info

# kedroの各種機能を使用するためのパッケージ達をインストール。私の環境では完了までに10分以上かかったので気長に待ってください
poetry add "kedro[all]"
```

# プロジェクト作成
kedro公式が事前に用意してくれている、Kedroを用いてIrisデータのロジスティック回帰を行うテンプレートファイルがあるので、それらを使って新規でプロジェクトを作成します。
以下のコマンドを実行すると、プロジェクト設定のRepository Nameで指定した名称のフォルダが、中にテンプレートがある状態で作成されます。

```shell

# 新規プロジェクト作成
poetry run kedro new
#以下、インタラクティブにプロジェクト設定をしていきます

#Project Name:
Getting Started
#Repository Name:
getting-started
#Python Package Name:
iris_test
#Generate Example Pipeline:
y
```

以上を行うと、このような構成のフォルダが作成されます。

```shell
getting-started
├── .coveragerc
├── .gitignore # Prevent staging of unnecessary files to git
├── .ipython # IPython startup scripts
│   └── profile_default
│       └── startup
│           └── 00-kedro-init.py
├── .isort.cfg
├── .kedro.yml # Path to discover project context
├── README.md # Project README
├── conf # Project configuration files
│   ├── README.md
│   ├── base
│   │   ├── catalog.yml
│   │   ├── credentials.yml
│   │   ├── logging.yml
│   │   └── parameters.yml
│   └── local
├── data # Local project data (not committed to version control)
│   ├── 01_raw
│   │   └── iris.csv
│   ├── 02_intermediate
│   ├── 03_primary
│   ├── 04_feature
│   ├── 05_model_input
│   ├── 06_models
│   ├── 07_model_output
│   └── 08_reporting
├── docs # Project documentation
│   └── source
│       ├── conf.py
│       └── index.rst
├── kedro_cli.py # A collection of Kedro command line interface (CLI) commands
├── logs # Project output logs (not committed to version control)
│   └── journals
├── notebooks # Project related Jupyter notebooks
├── setup.cfg
└── src # Project source code
    ├── iris_test
    │   ├── __init__.py
    │   ├── nodes
    │   │   └── __init__.py
    │   ├── pipeline.py
    │   ├── pipelines
    │   │   ├── __init__.py
    │   │   ├── data_engineering
    │   │   │   ├── README.md
    │   │   │   ├── __init__.py
    │   │   │   ├── nodes.py
    │   │   │   └── pipeline.py
    │   │   └── data_science
    │   │       ├── README.md
    │   │       ├── __init__.py
    │   │       ├── nodes.py
    │   │       └── pipeline.py
    │   └── run.py
    ├── requirements.txt
    ├── setup.py
    └── tests
        ├── __init__.py
        ├── pipelines
        │   └── __init__.py
        └── test_run.py
```
フォルダ群の内、dataフォルダとlogsフォルダの中身は`.gitignore`によってgitに反映されないようになっています。元から入っている
getting-started/data/01_raw/iris.csvだけはignoreされないので、不要な場合は適宜対処してください。

# vscodeでのデバッグ実行の準備
以下の例は、poetry導入時に`poetry config virtualenvs.in-project true`を実行し、プロジェクトディレクトリ内に.venvディレクトリが作成されるように設定した場合の方法です。この例では最終的に以下のフォルダ構成となります。

```shell

{poetry run kedro newを行ったディレクトリ}
├── .env
├── .venv/...
├── .vscode
        ├── launch.json
        └── settings.json
├── getting-started/...
├── poetry.lock
└── pyproject.toml
  
```

### .envファイルの作成

```'.env'
PYTHONPATH=/{作成したプロジェクトのsrcフォルダまでのPATH}:$PYTHONPATH
```

### launch.jsonの編集
python用のlaunch.jsonを開き、以下を追記。

```launch.json
   // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Kedro Run",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/getting-started/src/iris_test/run.py",
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}/getting-started"
        }
    ]
```

### settings.jsonの編集
```settings.json
{
  "python.pythonPath": ".venv/bin/python"
}
```

### デバッグ実行
`poetry run kedro new`を行ったディレクトリ(.venvがあるディレクトリ)でvscodeを開けばデバッグ実行できるはずです。


# サンプルコードを動かしてみる

Kedroのプロジェクトは大きく4つのコンポーネントから構成されます。

| Component  |説明 |
|---|---|
|Data Catalog |Pipelineの構築に使用されるデータセットの集合。それぞれのデータセットは`load`と`save`の能力を持ち、例えばpandas.CSVDataSetはCSVファイルをロードしたり保存できる。 |
| Pipeline |ノードの集合。nodeの依存関係や実行順序を管理する。  |
| Node|前処理や学習といった処理本体|
| Runner |pipelineを指定されたdata catalogを使って実行するもの。現在SequentialRunner,ParallelRunner,ThreadRunnerの3つがある。|

プロジェクト作成の際に`Generate Example Pipeline:y`としたので、Irisデータセットでロジスティック回帰を行う簡単なパイプラインが既に実装されています。まずはこれを動かして、全体の構成を見てみましょう。


```shell

#getting-started直下にて
poetry run kedro run
```

処理が完了したら`getting-started/logs/info.log`を見てください。行われた処理のログが出ています。

```info.log

2020-06-07 23:14:40,494 - root - INFO - ** Kedro project iris_test
2020-06-07 23:14:42,602 - kedro.io.data_catalog - INFO - Loading data from `example_iris_data` (CSVDataSet)...
2020-06-07 23:14:42,606 - kedro.io.data_catalog - INFO - Loading data from `params:example_test_data_ratio` (MemoryDataSet)...
2020-06-07 23:14:42,606 - kedro.pipeline.node - INFO - Running node: split_data([example_iris_data,params:example_test_data_ratio]) -> [example_test_x,example_test_y,example_train_x,example_train_y]
2020-06-07 23:14:42,612 - kedro.io.data_catalog - INFO - Saving data to `example_train_x` (MemoryDataSet)...
2020-06-07 23:14:42,613 - kedro.io.data_catalog - INFO - Saving data to `example_train_y` (MemoryDataSet)...
2020-06-07 23:14:42,613 - kedro.io.data_catalog - INFO - Saving data to `example_test_x` (MemoryDataSet)...
2020-06-07 23:14:42,613 - kedro.io.data_catalog - INFO - Saving data to `example_test_y` (MemoryDataSet)...
2020-06-07 23:14:42,614 - kedro.runner.sequential_runner - INFO - Completed 1 out of 4 tasks
2020-06-07 23:14:42,614 - kedro.io.data_catalog - INFO - Loading data from `example_train_x` (MemoryDataSet)...
2020-06-07 23:14:42,614 - kedro.io.data_catalog - INFO - Loading data from `example_train_y` (MemoryDataSet)...
2020-06-07 23:14:42,614 - kedro.io.data_catalog - INFO - Loading data from `parameters` (MemoryDataSet)...
2020-06-07 23:14:42,614 - kedro.pipeline.node - INFO - Running node: train_model([example_train_x,example_train_y,parameters]) -> [example_model]
2020-06-07 23:14:42,932 - kedro.io.data_catalog - INFO - Saving data to `example_model` (MemoryDataSet)...
2020-06-07 23:14:42,932 - kedro.runner.sequential_runner - INFO - Completed 2 out of 4 tasks
2020-06-07 23:14:42,932 - kedro.io.data_catalog - INFO - Loading data from `example_model` (MemoryDataSet)...
2020-06-07 23:14:42,933 - kedro.io.data_catalog - INFO - Loading data from `example_test_x` (MemoryDataSet)...
2020-06-07 23:14:42,933 - kedro.pipeline.node - INFO - Running node: predict([example_model,example_test_x]) -> [example_predictions]
2020-06-07 23:14:42,933 - kedro.io.data_catalog - INFO - Saving data to `example_predictions` (MemoryDataSet)...
2020-06-07 23:14:42,934 - kedro.runner.sequential_runner - INFO - Completed 3 out of 4 tasks
2020-06-07 23:14:42,934 - kedro.io.data_catalog - INFO - Loading data from `example_predictions` (MemoryDataSet)...
2020-06-07 23:14:42,934 - kedro.io.data_catalog - INFO - Loading data from `example_test_y` (MemoryDataSet)...
2020-06-07 23:14:42,934 - kedro.pipeline.node - INFO - Running node: report_accuracy([example_predictions,example_test_y]) -> None
2020-06-07 23:14:42,934 - iris.pipelines.data_science.nodes - INFO - Model accuracy on test set: 100.00%
2020-06-07 23:14:42,935 - kedro.runner.sequential_runner - INFO - Completed 4 out of 4 tasks
2020-06-07 23:14:42,935 - kedro.runner.sequential_runner - INFO - Pipeline execution completed successfully.
```
これだけでは何もわからないので、1つ1つ処理を追っていきます。

### kedro run
`kedro run`を実行すると、`src/iris_test/run.py`がキックされます。
その際に行われる処理は以下の流れになっています。

1. Kedroが提供するload_package_contextモジュールを用いて、project_contextインスタンスを作成する。この際、後述する`catalog.yml`や`logging.yml`、`parameters.yml`ファイルを読み込み、`DataCatalog`もインスタンス化する。
2. `Pipeline`をインスタンス化する
3. `SequentialRunner `クラスをインスタンス化し、`Pipeline`オブジェクトと`DataCatalog`オブジェクトを渡す。

まずパーツを作り、`SequentialRunner `クラスに渡して実行、といった流れのようです。各パーツについて見ていきます。

### DataCatalog
DataCatalogを定義する方法は2種類あり、1つは今実行したテンプレートでも用いている`catalog.yml`ファイルを使って設定を行うやり方、もう一つはAPIを使って設定するやり方があります。

#### 1.`catalog.yml`ファイルを使って設定を行う

```conf/base/catalog.yml
# Example 1: Loads / saves a CSV file from / to a local file system
example_iris_data:
  type: pandas.CSVDataSet
  filepath: data/01_raw/iris.csv
```
このように、最低限**データセット名(example_iris_data)、データタイプ(pandas.CSVDataSet)とファイルのありか(data/01_raw/iris.csv)**を設定すれば、読み込んでDataCatalogインスタンスを作成してくれるようです。
これ以外にも色々な設定ができ、いくつか例を示すと、
##### S3からCSVデータを読み込んで利用する設定

```conf/base/catalog.yml
# Example 4: Loads a CSV file from a specific S3 bucket, using credentials and load arguments
motorbikes:
  type: pandas.CSVDataSet
  filepath: s3://your_bucket/data/02_intermediate/company/motorbikes.csv
  credentials: dev_s3
  load_args:
    sep: ','
    skiprows: 5
    skipfooter: 1
    na_values: ['#NA', NA]
```

##### GCS上のエクセルファイルを読み込む設定
```conf/base/catalog.yml
# Example 6: Loads an excel file from Google Cloud Storage
rockets:
  type: pandas.ExcelDataSet
  filepath: gcs://your_bucket/data/02_intermediate/company/motorbikes.xlsx
  fs_args:
    project: my-project
  credentials: my_gcp_credentials
  save_args:
    sheet_name: Sheet1

```



##### DBにクエリを発行して得たデータを利用する設定

```conf/base/catalog.yml
# Example 12: Load a SQL table with credentials, a database connection, and applies a SQL query to the table
scooters_query:
  type: pandas.SQLQueryDataSet
  credentials: scooters_credentials
  sql: select * from cars where gear=4
  load_args:
    index_col: [name]
```
...などなど、色々と柔軟に設定できるようです。
データの取得には`fsspec`というPythonのファイルシステムインターフェースモジュールを使っていて、

- ローカルファイルシステム
- Hadoop File System (HDFS)
- Amazon S3
- Google Cloud Storage
- HTTP(s): http:// or https:// for reading data directly from HTTP web servers.

からデータを取り込むことができます。
またデータの読み込み/保存の際の区切り文字を何にするか、日付のフォーマット、圧縮するかしないか等々もここで指定できます。

##### バージョンコントロール
例えば時系列データを使用していて、今日から見て過去30日間のデータをinputとしたい時などがあると思いますが、以下のように設定することで日時をファイル名に入れたファイルを作成でき、実行時にソースのバージョンを指定して実行できるようです。

```conf/base/catalog.yml
cars.csv:
  type: pandas.CSVDataSet
  filepath: data/01_raw/company/cars.csv
  versioned: True

# data/01_raw/company/cars.csv/<version>/cars.csvのようにバージョン別に管理されて保存される
```

```shell
# 実行
kedro run --load-version="cars.csv:YYYY-MM-DDThh.mm.ss.sssZ"
```




DataCatalogの指定の仕方やバージョンコントロールの詳細は公式サイトの[DataCatalogのページ](https://kedro.readthedocs.io/en/stable/04_user_guide/04_data_catalog.html)をご覧ください。

#### 2.Code APIを使って設定を行う
.pyファイル中でDataCatalogの設定をする場合のやり方です。
いまいち使い所がわからない(ymlファイルでの指定で済むならそっちのほうが楽では？)のですが、Jupyterを使ってプロトタイピングしている時はこっちを使う感じでしょうか？
データをメモリに保存する処理なんかはJupyter等でプロトタイピングしている時でないとオススメできないと公式ドキュメントに記載されていたので恐らくCode API自体がそういう用途なのだと思うのですが...


```catalog.py
from kedro.io import DataCatalog
from kedro.extras.datasets.pandas import (
    CSVDataSet,
    SQLTableDataSet,
    SQLQueryDataSet,
    ParquetDataSet,
)
from kedro.io import MemoryDataSet

# DataCatalogの作成
io = DataCatalog(
    {
        "bikes": CSVDataSet(filepath="../data/01_raw/bikes.csv"),
        "cars": CSVDataSet(
            filepath="../data/01_raw/cars.csv", load_args=dict(sep=",")
        ),
        "cars_table": SQLTableDataSet(
            table_name="cars", credentials=dict(con="sqlite:///kedro.db")
        ),
        "scooters_query": SQLQueryDataSet(
            sql="select * from cars where gear=4",
            credentials=dict(con="sqlite:///kedro.db"),
        ),
        "ranked": ParquetDataSet(filepath="ranked.parquet"),
    }
)
# 利用できるデータソースを一覧表示する
io.list()

# DataCatalogに登録したデータの呼び出し
cars = io.load("cars")  # data is now loaded as a DataFrame in 'cars'
gear = cars["gear"].values

# データをメモリに保存する
memory = MemoryDataSet(data=None)
io.add("cars_cache", memory)
io.save("cars_cache", "Memory can store anything.")
io.load("car_cache")
```

### PipelineとNode
使用するパイプラインの定義は、`src/iris_test/pipeline.py`に書いていきます。
Irisデータセットを使ったテンプレートのパイプラインは以下のように記載されています。


```src/iris_test/pipeline.py

def create_pipelines(**kwargs) -> Dict[str, Pipeline]:
    """Create the project's pipeline.

    Args:
        kwargs: Ignore any additional arguments added in the future.

    Returns:
        A mapping from a pipeline name to a ``Pipeline`` object.

    """

    data_engineering_pipeline = de.create_pipeline()
    data_science_pipeline = ds.create_pipeline()

    return {
        "de": data_engineering_pipeline,
        "ds": data_science_pipeline,
        "__default__": data_engineering_pipeline + data_science_pipeline,
    }
```
`data_engineering_pipeline`と`data_science_pipeline`の2つのパイプラインがあり、デフォルト実行時は`data_engineering_pipeline`→`data_science_pipeline`の順で実行されます。

#### data_engineering_pipeline
data_engineering_pipelineで行われる処理は`src/iris_test/pipelines/data_engineering/pipeline.py`に書いていきます。

```src/iris_test/pipelines/data_engineering/pipeline.py
from kedro.pipeline import Pipeline, node
from .nodes import split_data

def create_pipeline(**kwargs):
    return Pipeline(
        [
            node(
                split_data,
                ["example_iris_data", "params:example_test_data_ratio"],
                dict(
                    train_x="example_train_x",
                    train_y="example_train_y",
                    test_x="example_test_x",
                    test_y="example_test_y",
                ),
            )
        ]
    )
```
create_pipelineの戻り値として、nodeが入った配列を引数に入れたPipelineを返却します。
nodeの引数には**当該nodeで実行させたい関数(split_data)、関数の引数、返却値**を指定します。

##### nodeで実行させたい関数
各ノードで実行させたい関数の中身は`src/iris_test/pipelines/data_engineering/nodes.py`に書いていきます。
pandas.dataframeとtrain/testの比率を受け取り、比率通りに分割したdictを返却します。
テンプレートでは単純に分割を行っただけですが、モデル開発を行う際は前処理のノードを加えて処理することになると思います。

```src/iris_test/pipelines/data_engineering/nodes.py
from typing import Any, Dict
import pandas as pd

def split_data(data: pd.DataFrame, example_test_data_ratio: float) -> Dict[str, Any]:
    """Node for splitting the classical Iris data set into training and test
    sets, each split into features and labels.
    The split ratio parameter is taken from conf/project/parameters.yml.
    The data and the parameters will be loaded and provided to your function
    automatically when the pipeline is executed and it is time to run this node.
    """
    data.columns = [
        "sepal_length",
        "sepal_width",
        "petal_length",
        "petal_width",
        "target",
    ]
    classes = sorted(data["target"].unique())
    # One-hot encoding for the target variable
    data = pd.get_dummies(data, columns=["target"], prefix="", prefix_sep="")

    # Shuffle all the data
    data = data.sample(frac=1).reset_index(drop=True)

    # Split to training and testing data
    n = data.shape[0]
    n_test = int(n * example_test_data_ratio)
    training_data = data.iloc[n_test:, :].reset_index(drop=True)
    test_data = data.iloc[:n_test, :].reset_index(drop=True)

    # Split the data to features and labels
    train_data_x = training_data.loc[:, "sepal_length":"petal_width"]
    train_data_y = training_data[classes]
    test_data_x = test_data.loc[:, "sepal_length":"petal_width"]
    test_data_y = test_data[classes]

    # When returning many variables, it is a good practice to give them names:
    return dict(
        train_x=train_data_x,
        train_y=train_data_y,
        test_x=test_data_x,
        test_y=test_data_y,
    )

```

##### nodeで実行させたい関数の引数
nodeに渡したいDataCatalogに登録したデータセット名(example_iris_data)やパラメータ(params:example_test_data_ratio)を指定できます。パラメータは、`params:`という接頭辞をつけると、`parameters.yml`に記載したパラメータを読み込んでくれます。

```conf/base/parameters.yml
example_test_data_ratio: 0.2
example_num_train_iter: 10000
example_learning_rate: 0.01
```
このパラメータは階層構造を作ることができ、以下のようにアクセスが可能です。

```python

# parameters.ymlファイル
step_size: 1
model_params:
    learning_rate: 0.01
    test_data_ratio: 0.2
    number_of_train_iterations: 10000

# node関数定義
def train_model(data, model):
    lr = model["learning_rate"]
    test_data_ratio = model["test_data_ratio"]
    iterations = model["number_of_train_iterations"]
    ...

# in pipeline definition
node(
    func=train_model,
    inputs=["input_data", "params:model_params"],
    outputs="output_data",
)
```
また、単純に"parameters"と書いて全てのパラメータを渡すことも可能です。

```python

# parameters.yml
def increase_volume(volume, params):
    step = params["step_size"]
    return volume + step

# in pipeline definition
node(
    func=increase_volume, inputs=["input_volume", "parameters"], outputs="output_volume"
)
```



#### data_science_pipeline
こちらも同じくpipeline定義を見ていきます。こちらはtrain_model,predict,report_accuracyの3つのノードを実行します。
ノードの中身は`src/iris_test/pipelines/data_science/nodes.py`にありますが、少々長く、特別なことはしていないので省略します。

```src/iris_test/pipelines/data_science/pipeline.py
from kedro.pipeline import Pipeline, node
from .nodes import predict, report_accuracy, train_model

def create_pipeline(**kwargs):
    return Pipeline(
        [
            node(
                train_model,
                ["example_train_x", "example_train_y", "parameters"],
                "example_model",
            ),
            node(
                predict,
                dict(model="example_model", test_x="example_test_x"),
                "example_predictions",
            ),
            node(report_accuracy, ["example_predictions", "example_test_y"], None),
        ]
    )
```

### Runner
DataCatalogインスタンスとPipelineインスタンスができたら、いよいよRunnerに渡して処理を実行します。現時点で使用できるRunnerは3種類あります。まず、テンプレートで使用されているSequentialRunnerから見ていきます。

#### SequentialRunner
順次ノードを実行していくrunnerです。`kedro run`に何もオプションを付けずに実行するとSequentialRunnerで処理が実行されます。

#### ParallelRunner
ノードを並列に実行していくrunnerです。`kedro run --parallel`とオプションを付けるとParallelRunnerで処理が実行されます。
”並列に”というのは、まずPipelineの各ノードの依存関係を読み込み、トポロジカルソートを使って並べ替え、並列化できるところ(依存関係がないところ)は同時に実行してくれるみたいです。例えば複数モデルのアンサンブルをするようなパイプラインの場合はこちらで実行すれば処理が早くなりそうです。

#### ThreadRunner
公式ドキュメントにもあまり多くの記述が無くよくわからなかったのですが、Apache Sparkを使用するnodeを組み込んだpipelineを実行する場合、ParallelRunnerを使うと実行順序がおかしくなる可能性があるため、ThreadRunnerを使用する必要があるそうです。`kedro run --runner=ThreadRunner`で実行できます。


## 終わりに
少々長くなってしまいました。今回は単純な処理を行う場合の手法を書きましたが、kaggleで使う時、実際のプロダクトに組み込む時のTips等、知見が溜まったら書いていきたいなと思います。
