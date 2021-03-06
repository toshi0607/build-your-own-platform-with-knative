# Cloud Run

ここまではGKEのKubernetesクラスタにKnativeのコンポーネントを自分でインストールし、プラットフォームを構築してきました。Cloud RunはKubernetesクラスタも意識することなくコンテナを実行できるKnative互換APIに基づくマネージドサービスです。

## Hello World

早速Cloud Runをセットアップし、ServingのHello Worldワークショップで利用したhelloworld-goを実行してみましょう。

```shell
# Cloud Run APIの有効化
$ gcloud services enable run.googleapis.com

# Cloud Runに関するご自身のGCPプロジェクトのデフォルトを設定
$ gcloud config set run/platform managed
$ gcloud config set run/region asia-northeast1

# コンテナイメージをCloud Runにデプロイ
$ gcloud run deploy --image gcr.io/knative-samples/helloworld-go
# プロンプト
## Service Name (helloworld-go): 何も入力せずEnter
## Allow unauthenticated invocations to [helloworld-go]: y

Deploying container to Cloud Run service [helloworld-go] in project [<your-project-id>] region [asia-northeast1]
✓ Deploying new service... Done.             
  ✓ Creating Revision...       
  ✓ Routing traffic...         
  ✓ Setting IAM Policy...      
Done.                                                                                                                        
Service [helloworld-go] revision [helloworld-go-xxxxx] has been deployed and is serving 100 percent of traffic at https://helloworld-go-yyyyyyyyyy-an.a.run.app
```

リクエストしてみましょう。

```shell
# 発行されたURLで置き換えてください
$ export HELLO_WORLD_GO_URL=https://helloworld-go-<your hello world go URL>-an.a.run.app
$ curl $HELLO_WORLD_GO_URL
Hello World!
```

環境変数をセットしてリクエストし直してみましょう。

```shell
$ gcloud run services update helloworld-go --update-env-vars TARGET="Cloud Run!"
$ curl $HELLO_WORLD_GO_URL
Hello Cloud Run!
```

`hey`を利用して少し負荷をかけてからメトリクスを確認してみましょう。

```shell
# 10秒間5並列でリクエスト
$ hey -z 10s -c 5 $HELLO_WORLD_GO_URL
```

つぎのURLからメトリクスをはじめ、変更毎にRevisionが作成されている様子やログ、Knative Serviceのマニフェストなどが確認できます。

https://console.cloud.google.com/run/detail/asia-northeast1/helloworld-go/metrics

step1と同様にトラフィック分割を試してみましょう。

トラフィックを流すRevision名はコンソールから確認してください。

https://console.cloud.google.com/run/detail/asia-northeast1/helloworld-go/revisions

```shell
# Revision名はご自身のものに置き換えてください
$ gcloud alpha run services update-traffic --region=asia-northeast1 --platform=managed --to-revisions=helloworld-go-00002-xob=50,helloworld-go-00001-yul=50
$ curl $HELLO_WORLD_GO_URL
```

Cloud Runは2019年11月にGAされたリリースですが、現状Knative ServingのAPIすべてに互換性があるわけではなく、順次機能追加されています。コンテナベースのアプリケーションをメインで扱っていなくても、既存のコンテナをサーバーレスなAPIにするなど触れ合う機会は増えていくかもしれません。

また、GKEやオンプレミス、マルチクラウド環境などでCloud Runを構築・実行する[Cloud Run for Anthos](https://cloud.google.com/run/docs/gke/setup)もあります。

GCPのサーバーレスソリューション

![](./images/gcp_serverless_options.png)
https://cloud.google.com/serverless-options/

## 参考

* [Cloud Run、Cloud Run for Anthosの機能比較](https://cloud.google.com/run#choose-the-platform-that-fits-you)
* [Cloud Run on GKEに覗くKnative](https://qiita.com/toshi0607/items/eeeabe81b1beac343b6b)
* [Anthos](https://cloud.google.com/anthos/)
* [Mastering Serverless Applications with Google Cloud Run](https://learning.oreilly.com/library/view/mastering-serverless-applications/9781492057086/)
  * Early Release段階ですが、Cloud FunctionsやApp Engineとの使い分けや、ServerlessなコンピューティングソリューションをFunctionとServiceに分類するなど視点が面白く感性が期待されます。

[戻る](step3.md) | [次へ](step5.md)
