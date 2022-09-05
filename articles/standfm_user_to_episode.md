---
title: "放送レコメンドを機能をTensorFlow Recommendersで作り、ABテストしてみた"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python","ML","TensorFlowRecommenders","VertexPipeline","FirebaseABTesting"]
published: True
---

# はじめに

こんにちは、stand.fm でMLエンジニアをしているcanonrockです。

stand.fmでは各画面に表示するコンテンツのパーソナライズを進めており、先日その一環としてホーム画面に各ユーザーへおすすめの放送を表示する機能を追加しました。本記事ではおすすめ放送表示機能のシステム構成やABテスト結果をご紹介しようと思います。



# おすすめ放送表示機能の目的と概要
### 概要

stand.fmではアプリを開いた時に最初に出る画面をホーム画面と呼んでいます。ホーム画面では画像のようにテーマ別にセクションが並び、セクション内に放送がいくつか表示されます。

![image](/images/standfm_user_to_episode/IMG_7644.png =300x)

ホーム画面に表示されるセクションや各セクションに表示される内容はパーソナライズされておらず、全ユーザーに対して同一の内容が表示される状態でした。しかし、やはりユーザーによって嗜好が異なるので、各ユーザーが好きそうな放送をホーム画面に出したいニーズがあります。そこで、パーソナライズ第一弾として、各ユーザーの過去の視聴履歴から各ユーザーが好みそうな放送を抽出し、まとめて表示する「あなたへのおすすめ放送」セクションを追加することにしました。



### 新機能で向上させたいKPI

本セクション追加にあたり、最初に本セクションによって向上させたいKPIを定め、リリース後のABテストによってKPIの推移を確認することとしました。KPIは以下にしました。

- ホーム画面からの放送再生数
- 新規フォロー数
  - stand.fmでは、youtubeのチャンネル登録のようにチャンネルをフォローする機能がついています。嗜好に合う未知のチャンネルのエピソードに触れることで、新しくチャンネルをフォローしてくれるのでは？ということで指定しました。
- アプリ滞在時間



# システム構成

### 全体像

![image-20220823175030777](/images/standfm_user_to_episode/image-20220823175030777.png)

大まかに、Vertex Pipelinesで作成したパイプラインを週に一回キックしてモデルを再学習し、AI Platform Predictionでサービング、それをバックエンドサーバーから叩いてクライアントアプリにレコメンド結果を渡すという流れになっています。



### ABテスト周り

ABテストで必要な群の分割とそれぞれの群で使用するconfig値の取得はFirebase A/BTestingを利用しています。

##### 群の分割とconfig値の受け渡し

群の分割はABテスト開始後にFirebase A/BTestingが自動でやってくれ、ユーザーと群の対応はBigqueryにexportされるので、ABテスト集計時に使用できます。

Firebase A/BTestingは裏でFirebase Remote Configを使用しており、それぞれの群で参照させたいAI Platform Prediction上でサービングされているモデルのモデル名をconfig値として設定しておけば、アプリからモデル名を取得し、バックエンドAPIコール時に渡せるようになります。AI Platform Predictionはモデルのバージョン管理に対応しており、モデルのコール時にバージョン指定をしなかった場合はモデルごとに設定されたデフォルトのバージョンが使用され、バージョン指定時は指定したバージョンのモデルが使用されます。今回のバックエンド側実装ではバージョン指定あり/なしのモデル名、もしくは空文字を受け取れるようにしておき、

- 週一回の再学習で更新された最新バージョンのモデルを使いたい時:バージョン指定なし

- 週一回の再学習を無視して常に特定のバージョンのモデルを使いたい時:バージョン指定あり
- その群では「あなたへのおすすめ放送」セクションを表示したくない時:空文字

のように使い分けができるようにしました。



##### ABテスト期間中のKPI推移の観察

Firebase A/BTestingではABテスト開始時に追跡するeventを6つまで指定することができ、指定したeventについてはABテスト期間中に集計が行われ、コンソール画面上でそれぞれの群の結果をグラフで表示することができます。

![image-20220823182019474](/images/standfm_user_to_episode/image-20220823182019474.png)

上記画像の例では、ベースライン(「あなたへのおすすめ放送」セクション非表示群)とVariant A(「あなたへのおすすめ放送」セクション表示群)のABテスト期間中のチャンネルフォロー率の推移が表示されています。

このようにABテスト期間中に手軽に速報値を確認できるのは便利なのですが、いくつか注意点があります。Firebaseの制約としてユーザーをFirebase install IDでしか認識できないというものがあり、このFirebase install IDはアプリの再インストールによって変わってしまうため、同一ユーザーが複数群に振り分けられてしまう場合があります。また、ABテスト期間中に何らかのエラーによって新機能を利用できなかったユーザー等は集計結果から除外したいですが、Firebaseのコンソール上の集計ではそういった対応はできません。このため、このコンソール上で見られる値はあくまで速報値として利用し、最終的な結果集計はBigquery等の別ツールで行うのが良いと思います。



##### Firebase A/BTestingの選定

A/Bテストの実現手法を検討する段階では以下のような状況でした。

- Firebaseはイベントロギング等の用途で導入済
- [LaunchDarkly](https://launchdarkly.com/features/experimentation/)というサービスのFeature Flag機能を利用中
- ログデータはBigqueryに自動連携され、Bigquery上で詳細な集計が可能。Redashも整備されている



上記を踏まえ、

1. Firebase A/BTesting
2. LaunchDarklyののABTesting機能
3. ABTest用のサービスは使わず、自前で群分割やconfig設定機構を作る

の選択肢を検討しました。



結果、

- Firebase A/BTestingなら無料で使えるが、LaunchDarklyはトラフィック量に応じて追加料金が発生する
  - 1M/monthのevent数あたり$126の料金で、当時のstand.fmでは1M~5M程度のトラフィック量を想定
  - トラフィック増加と共に料金も線形増加するため、下手にABTest施策を増やすと料金が高くなる
- Firebase A/BTestingなら既存のログで利用しているevent定義(チャンネルフォロー、エピソード再生...etc)を流用可能だが、LaunchDarklyでは1から定義を作り直す必要がある
- LaunchDarklyの方がFirebase A/BTestingと比べてサービス上で表示できるダッシュボードが充実していそうだが、結局エラーユーザー除外などを施した最終版の集計結果はRedashで可視化することになる
- 自前でABTest機構を作るのは工数がかかる・簡易的に作るとABTestの開始/終了の度にバックエンド側のデプロイが必要になって面倒

といった理由から、Firebase A/BTestingを選定しました。



### バックエンドでのレコメンド結果のフィルタリング

バックエンドではアプリから受け取ったモデル名に応じてAI Platform Prediction上のモデルを叩き、適宜レコメンド結果をフィルタリング・ランダム抽出して返却しています。

レコメンドモデルは後述するTensorFlow Recommendersを使用していますが、モデル側では視聴済み放送の除外や、ブロック関係にあるユーザーの放送の除外に対応できないため、バックエンド側でこれらのフィルタリングを施しています。(正確に言えばAI Platform Predictionのカスタムルーチン機能を使えばAI Platform Prediction側でこれらの対応は可能なのですが、モデルはレコメンドを生成するだけのシンプルなものにしておきたい点、視聴済み放送はリアルタイムに変化するためバックエンド側で処理した方が楽な点からバックエンド側での対応としました。)

その他、新規ユーザーのためレコメンドが生成できなかった、フィルタリングによって表示できる放送が無くなってしまった、AI Platform Predictionがタイムアウトしたといったエラーをロギングしておき、後のABテスト集計で利用できるようにしました。



### Cloud FunctionsとVertex Pipelinesを用いた週一回の自動再学習

モデルの再学習とサービングはVertex Pipelines+Cloud Functionsで自動化しています。

Vertex Pipelinesは外部ストレージからのデータの読み込み、前処理、モデル学習、デプロイ...といった一連の流れをパイプラインとしてまとめ、実行を管理できるサービスです。現時点でVertex Pipelines自体に定期実行の仕組みが無いため、Cloud Scheduler+Pub/Sub+Cloud Functionsで定期的に事前定義した関数を実行できるようにしておき、Vertex Pipelinesをキックするようにしています。Vertex Pipelines内ではBigqueryから最新データを読み込み、AI Platform Trainingを利用してモデルを学習しています。(AI Platform Trainingを使っているのは以前使用していたAI Platform Pipelinesの名残で、AI Platform PredictionのVertex Endpoint移行と合わせていずれは全てVertexに移行したいと思っています。)

ここらへんのMLOpsの仕組み構築は他にも色々と工夫している点があり、定期実行だけでなく単発でも簡単に実行できるようにしてあったり、Github Actionsと連携してmain/developブランチにマージしたら自動でprod/dev環境のVertex PipelineやCloud Functionsが更新されるようにしてあったり...と書きたいことがたくさんあるのですが、記事がかなり長くなってしまいそうなのでまた別記事で紹介できればと思います。



# おすすめ放送モデルの作成方法

おすすめ放送レコメンドモデルの構築には[TensorFlow Recommenders](https://www.tensorflow.org/recommenders)(以下TFRS)を利用しています。TFRSはチュートリアルが充実していて、データがあれば簡単にRetrieval(情報検索)モデル,Ranking(情報ランク)モデル,これらを組み合わせたマルチタスクモデルなどを構築することができます。

今回のおすすめ放送モデル作成では、Retrieval単独・Ranking単独・マルチタスクモデルで学習と評価を行い、最終的にマルチタスクモデルを選択しました。



##### inputデータの前処理

Retrievalモデルは古典的なMatrix Factarizationモデルに該当するモデルで、ユーザーがその放送を視聴した=1、視聴しなかった=0として評価値行列を作り、行列分解を行います。このためinputデータは「ユーザーID,放送ID」のような形式になります。

一方、Rankingモデルは各視聴履歴同士に順序が必要なので、何らかの形でスコアを付ける必要があります。stand.fmではユーザーが各放送を全体の何%視聴したかのログを取っているので、100%=1, 50%=0.5, 0%=0としてスコアにしました。inputデータは「ユーザーID,放送ID,スコア」の形になります。

マルチタスクモデルはRetrievalでもRankingでも使える形でデータを供給する必要があるので、inputデータは「ユーザーID,放送ID,スコア」の形になります。



##### マルチタスクモデルの実装

main関数の一部抜粋を以下に記載します。実際のコードでは前処理コードやらロギングコードやらが入りますが、モデル作成〜学習〜保存までの流れはシンプルに書けます。

注意点として、TFRSではmodelのfit時に`tf.distribute.Strategy`を使用することで複数GPUによる分散処理ができるのですが、マルチタスクモデルでは単純に指定しただけだとエラーが出てしまいました。何かしら解決策はありそうな気もしますが、現状は単一GPUで処理しています。

また、AI Platform Predictionを利用してモデルをサービングする場合、モデルにsignatureがついていないといけないのですが、ただモデルをfitさせただけだとsignatureが付きません。一度予測を行うことでsignatureが付与されるので、汚い手法ですがコード中で適当なユーザーに対して予測を行っています。

```python
# TODO: 並列処理に対応させる。普通にmirrored stratedyを指定しただけだとエラーになる
model = MultiTaskModel(
    unique_item_ids=unique_episode_ids,
    unique_user_ids=unique_user_ids,
    user_dict_key="user_id",
    item_dict_key="episode_id",
    rating_dict_key="score",
    embedding_dimension=embedding_dimension,
    rating_weight=rating_weight,
    retrieval_weight=retrieval_weight,
)
model.compile(optimizer=tf.keras.optimizers.Adamax(learning_rate))
model.fit(x=train, validation_data=test, epochs=max_epoch_num, callbacks=callbacks)

index = tfrs.layers.factorized_top_k.BruteForce(model.user_model, k=predict_num)
index.index_from_dataset(episodes.batch(100).map(lambda x: (x["episode_id"], model.item_model(x["episode_id"]))))

# データセット中のユーザーに対して予測を出力
# これを行わないと、何故か自動でsignatureが設定されず、AI platform predictionでのモデルのvalidationに通らない
# 手動でsignatureを付ける方法はいくつかあるが、どれもうまくいかなかったので、汚い手法であるがやむを得ずこうしている
# https://github.com/tensorflow/recommenders/issues/131
sample_user = unique_user_ids[0]
index(np.array([sample_user]))

# モデルをexportし、GCSに保存
tf.saved_model.save(index, model_save_path)
upload_folder(path=model_save_path, bucket_name=bucket_name)
```

マルチタスクモデル本体の実装は以下のように書いています。

注意点として、Retrievalモデルの評価タスクは下記コードの64行目のようにtraining中は行わない設定にしておくことを強くおすすめします。

`tfrs.tasks.Retrieval`のタスクはGPUを使わずに行われ、かなりの時間がかかります。これをtraining中も行う設定にしておくと、GPUを使えるtraining処理自体にかかる時間<<<<`tfrs.tasks.Retrieval`のタスクにかかる時間になってしまい、外からGPU利用率を監視するとまるでGPUが正常に動いていないかのような挙動に見えます。自分はこれで結構な時間を溶かしました...

また、今回のデータではembedding_dimensionは大きければ大きいほど良いレコメンドになる傾向があったため、GPUが許す限り大きい値に指定するのが良さそうでした。ただ、あまりembedding_dimensionを上げすぎるとモデルのレイテンシも悪化するため、トレードオフとなります。

```python
class MultiTaskModel(tfrs.models.Model):
    def __init__(
        self,
        unique_item_ids,
        unique_user_ids,
        embedding_dimension,
        rating_weight: float,
        retrieval_weight: float,
        user_dict_key,
        item_dict_key,
        rating_dict_key,
    ) -> None:
        super().__init__()

        self.user_model = tf.keras.Sequential(
            [
                tf.keras.layers.StringLookup(vocabulary=unique_user_ids, mask_token=None, num_oov_indices=0),
                tf.keras.layers.Embedding(len(unique_user_ids), embedding_dimension),
            ]
        )
        self.item_model = tf.keras.Sequential(
            [
                tf.keras.layers.StringLookup(vocabulary=unique_item_ids, mask_token=None, num_oov_indices=0),
                tf.keras.layers.Embedding(len(unique_item_ids), embedding_dimension),
            ]
        )
        self.rating_model = tf.keras.Sequential(
            [
                tf.keras.layers.Dense(256, activation="relu"),
                tf.keras.layers.Dense(128, activation="relu"),
                tf.keras.layers.Dense(1),
            ]
        )

        self.rating_task: tf.keras.layers.Layer = tfrs.tasks.Ranking(
            loss=tf.keras.losses.MeanSquaredError(), metrics=[tf.keras.metrics.RootMeanSquaredError()]
        )

        self.retrieval_task = tfrs.tasks.Retrieval(metrics=None)

        self.user_dict_key = user_dict_key
        self.item_dict_key = item_dict_key
        self.rating_dict_key = rating_dict_key
        # The loss weights.
        self.rating_weight = rating_weight
        self.retrieval_weight = retrieval_weight

    def call(self, features: Dict[Text, tf.Tensor]) -> tf.Tensor:
        user_embeddings = self.user_model(features[self.user_dict_key])
        item_embeddings = self.item_model(features[self.item_dict_key])

        return (
            user_embeddings,
            item_embeddings,
            self.rating_model(tf.concat([user_embeddings, item_embeddings], axis=1)),
        )

    def compute_loss(self, features: Dict[Text, tf.Tensor], training=False) -> tf.Tensor:

        ratings = features.pop(self.rating_dict_key)
        user_embeddings, item_embeddings, rating_predictions = self(features)

        rating_loss = self.rating_task(labels=ratings, predictions=rating_predictions, compute_metrics=not training)
        retrieval_loss = self.retrieval_task(user_embeddings, item_embeddings, compute_metrics=not training)

        return self.rating_weight * rating_loss + self.retrieval_weight * retrieval_loss
```



##### 視聴済みの放送が多くレコメンド上位に来てしまう問題

TFRSで作成するモデルは過去の視聴履歴からユーザーが好みそうな放送をランク付けして出力してくれますが、その際にその放送が視聴済みかどうかの判断はしてくれません。どうしても出力順上位の放送は実際にユーザーが過去に視聴した放送が多くなってしまうため、実際にアプリに表示したいレコメンドが30件程度だったとしても、モデル側では200件程度出力しておき、そこから視聴済み放送をフィルタリングするといった処理が必要になります。

stand.fmで実際におすすめ放送のレコメンドを開始してみたら、モデル側で200件出力していてもその殆どが視聴済みで、出せるおすすめ放送が無くなってしまうユーザーが多発してしまいました。解決策としては単純にモデルからの出力件数を増やせば良いのですが、レイテンシ悪化や、ランキング上位の放送が画面上に表示されづらくなることとの兼ね合いを見て対応していこうと思っています。



### 今後対応したい課題

##### 属性情報を全く考慮に入れていない問題

現状使用しているマルチタスクモデルでは、ユーザーやアイテムの属性情報は考慮されていません。解決策として、TFRSのチュートリアルでも紹介がある[DCN](https://www.tensorflow.org/recommenders/examples/dcn)への乗り換えを検討しています。DCNはいわゆるFactrization Machinesのようなことをするモデルで、各ユーザーやアイテムの特徴量のクロス項を用いてレコメンドを出力することができます。stand.fmでは放送に付与されたハッシュタグやカテゴリ等の属性データがあるので、これらの属性を付与することでより精度の高いレコメンドができるのではないかと期待しています。



##### 全体の10%のみ視聴した放送はポジティブな嗜好を示すのかネガティブな嗜好を示すのか問題

現状のマルチタスクモデルでは、全体の10%を視聴した放送のスコアは0.1としてinputしています。しかし、全体の10%を視聴したということは、全体の10%のみを視聴して視聴するのをやめてしまったとも言えるので、必ずしもポジティブな嗜好を示すとは限らないように思います。このため、視聴の進行度合いが一定以上の視聴履歴のみをinputに入れるように閾値を設けることを検討しています。

ただ、放送全体の長さが1時間の放送を10%視聴した場合と、全体が5分の放送を10%視聴した場合では、前者の方がユーザーはその放送を気に入ったと考えることが妥当に思えます。このため、スコアの作成方法自体も絶対時間で作るように見直すことも考えています。



##### impressionデータも利用したい

stand.fmでは、「ユーザーがその放送を画面内で表示した」という状況を示すimpressionデータを取得しています。

ユーザーがある放送を何度も画面内で表示しているにも関わらず、その放送を視聴していない場合、ユーザーはその放送を好んでいない可能性があります。このような放送に相対的に低いスコアを付けることでより精度の高いレコメンドになったりしないだろうかと考えており、今後検証していくつもりです。



# ABテスト結果

ABテストは以下のような設定で行いました。

- 集計期間:機能リリース一週間後から二週間(リリース直後はノベルティ効果によって必要以上に新機能が利用される可能性が高いため除外)

- 集計対象ユーザー:集計期間中に何らかの形でイベントを発生させた(=期間中にアプリを利用した)ユーザー - A群B群両方に振り分けられたユーザー - レコメンド結果フィルタリングによっておすすめ放送セクションが表示されなかったユーザー
- A群:おすすめ放送セクションを出さないユーザー群,B群:おすすめ放送セクションを出すユーザー群
  - A群B群は集計期間中にアプリを利用したユーザーを50%ずつ振り分け



結果は以下でした。

##### 数値比較

- 放送再生数:B群で2.47%向上
- 放送へのいいね数:絶対数はB群で6.75%向上、ABテスト期間中に一度でもいいねした人のユニークユーザー数はB群で2.38%向上
- チャンネルフォロー数:絶対数はB群で1.52%向上、ABテスト期間中に一度でもチャンネルフォローした人のユニークユーザー数はB群で4.42%向上
- アプリ滞在時間:B群で0.2分向上(ほぼ変化なし)
- DAU:B群で1.24%向上

##### 平均値の差の検定

放送再生数、放送へのいいね数、チャンネルフォロー数、アプリ滞在時間、DAUの全てでp<0.05水準では有意差なし。

##### カイ二乗検定(ABテスト期間中に一度でも放送へのいいねやチャンネルフォローをしたか)

放送へのいいね:pvalue=0.26,チャンネルフォロー:pvalue=0.30で、p<0.05水準では有意差なし。



結論、数値上では向上効果は見られましたが、有意な差にはなりませんでした。。(残念)

ABテスト実施前の予想では放送の再生数は向上してもチャンネルフォローまでは影響しないだろうと思っていたのですが、数値比較の結果を見てみるとむしろチャンネルフォローしたユニークユーザー数や放送へのいいね数が増加していたのが意外でした。ユーザーが一日のアプリ利用時間内に視聴できる放送数は一定だが、レコメンドによってよりいいねしたくなるような嗜好に合致する放送を出せたり、今まで知らなかった嗜好に合うチャンネルに出会うきっかけを提供できたということなのでしょうか。

今後のモデル改善によって、有意差が出るくらい良いレコメンドができたらなと思います。



# 今後の課題

今後のおすすめ放送セクションの課題として、精度向上とレイテンシ・負荷改善があります。

精度向上については、モデル作成方法の下りで触れたように属性情報を入れるなどまだまだモデル自体に改善の余地があるので、モデルを進化させてより良いレコメンドにしていきたいと思っています。

##### レイテンシ・負荷改善

現在、AI Platform Predictionでサービングしているモデル部分だけで、50パーセンタイルで600~700ms,95パーセンタイルで1000ms程度のレイテンシが発生しています。実際にクライアントアプリにおすすめセクションが表示されるまでにはバックエンド側で行っているフィルタリング処理にかかる時間も乗ってくるので、更に+αで時間がかかります。

stand.fmのアプリでは遅延ローディングを採用しており、アプリ起動時にはおすすめ放送セクションのような取得に時間がかかるセクションは読み込まず、すぐに取得できるセクションのみロードしているのでユーザーの使い勝手が直接悪くなるわけでは無いのですが、今後おすすめセクションを初回ロードセクションに含めたくなった場合にはレイテンシ改善がマストになります。

また、AI Platform Prediction自体にも結構負荷がかかっているようで、ユーザーが集中する時間帯ではそれなりの頻度でAI Platform Predictionがタイムアウトしてしまいます。タイムアウト時にはおすすめ放送セクションが非表示になるようにしてはいますが、できれば負荷が高まっても常にセクション表示をしたいものです。AI Platform Predictionは複数ノードでの分散処理に対応しているので、ノード数を増やせばある程度の改善は見込めますが、当然その分サーバー費用もかさんでしまいます。

レイテンシ・負荷問題の解決策としてはScaNNという最近傍検索ライブラリの使用を予定しています。現在はモデルの最後の部分でレコメンドする放送をスコア順に並べ替える処理を`tfrs.layers.factorized_top_k.BruteForce`レイヤで行っていますが、これは愚直に全ての放送をスコア順に並べ替えるため、計算に時間がかかります。ここをScaNNレイヤに差し替えることで近似的に高いスコアの放送を取得できるようになり、レイテンシ・負荷問題の改善が見込めます。

ただ、素のAI Platform PredictionのノードではScaNNレイヤを扱えないため、カスタムコンテナでのサービングをするようにしたり、Vertex Endpointに乗り換える等の処置が必要になります。stand.fmでは徐々にAI PlatformからVertexへの乗り換えを進めているので、Vertex Endpointへの乗り換えタイミングでScaNNレイヤを使うように変更したいと思っています。



# おわりに

今回追加したおすすめ放送表示機能では、いいね数やチャンネルフォロー数の向上ができました。

今後の展望としては、放送ではなくチャンネルを推薦するセクションを追加したり、放送詳細画面で関連放送を出す、ホーム画面の構成自体をパーソナライズするなどの機能追加を予定しています。これらのパーソナライズでガンガンKPIを向上させていきたいですね！


・・・

株式会社 stand.fm では、絶賛エンジニアを募集しております！

募集職種はこちらから
https://corp.stand.fm/recruit?utm_source=standfm-web&utm_content=top

Meetyからカジュアル面談も募集しています。
https://meety.net/matches/JDMmSgfMhhfl


