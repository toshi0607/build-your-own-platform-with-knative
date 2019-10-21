# Knative Lambda Runtimes

[Knative Lambda Runtimes](https://github.com/triggermesh/knative-lambda-runtime)（KLR、クリアー）はKnative、TektonベースのOSSです。AWS Lambda互換のfunctionをKnative、TektonがインストールされたKubernetesクラスタで実行できます。

セットアップが済んだらKnariveで構築するFaaSプラットフォームの事例としてプラットフォーム利用者観点で見ていきましょう。

KLRは`tm`というコマンドで操作します。つぎのコマンドを実行して`tm`をインストールしてください。

```shell
$ go get github.com/triggermesh/tm
```

つぎにKLRがGoのLambda functionをビルドするための`Task`を利用できるようにします。つぎのコマンドを実行してください。

```shell
$ tm deploy task -f https://raw.githubusercontent.com/triggermesh/knative-lambda-runtime/master/go-1.x/runtime.yaml
```

コンテナイメージの保存はこのワークショップではGCRを利用します。GCRを利用するにあたってKubernetesの`Secret`オブジェクトの登録が必要です。つぎのコマンドを実行して一時的なアクセストークンを発行し、登録してください。

```shell
$ tm set registry-auth gcr-image-puller
# Registry: gcr.io/<your-project-id> ご自身のGCPプロジェクトIDに置き換えてください
# Username: oauth2accesstoken
# Password: <gcloud auth print-access-tokenを実行して得られる値>
```

いよいよfuncionをデプロイします。つぎのファイルを`klr`というフォルダの下に`main.go`という名前で保存し、`tm`コマンドを実行してデプロイしてください。

```go
package main

import (
        "fmt"
        "context"

        "github.com/aws/aws-lambda-go/lambda"
)

type MyEvent struct {
        Name string `json:"name"`
}

func HandleRequest(ctx context.Context, name MyEvent) (string, error) {
        return fmt.Sprintf("Hello %s!", name.Name ), nil
}

func main() {
        lambda.Start(HandleRequest)
}
```

```shell
$ tm deploy service go-lambda -f ./klr --build-template knative-go-runtime --registry-secret gcr-image-puller --wait
Uploading "." to go-lambda-q7dzp-pod-7881a3
Waiting for taskrun "go-lambda-q7dzp" ready state
Waiting for service "go-lambda" ready state
Service go-lambda URL: http://go-lambda.default.example.com
```

つぎのコマンドでfunctionが実行されるのを確認してください。

```shell
# export IP_ADDRESS=$(kubectl get svc istio-ingressgateway --namespace istio-system --output 'jsonpath={.status.loadBalancer.ingress[0].ip}')
$ curl -H "Host: go-lambda.default.example.com" http://$IP_ADDRESS --data '{"Name": "Foo"}'
```

## 参考

* [GCR Authentication methods](https://cloud.google.com/container-registry/docs/advanced-authentication)
* [triggermesh/tm](https://github.com/triggermesh/tm)
* [triggermesh/knative-lambda-runtime](https://github.com/triggermesh/knative-lambda-runtime)
* [triggermesh/aws-custom-runtime](https://github.com/triggermesh/aws-custom-runtime)
* [triggermesh/knative-local-registry](https://github.com/triggermesh/knative-local-registry)
* KLRのアーキテクチャーとCustom AWS Lambda Runtimes
  * [「Knativeで実現するKubernetes上のサーバーレスアーキテクチャ」の文字起こし #CNTD2019: ユースケース⑥　FaaS　イベントpull型](https://qiita.com/toshi0607/items/6dbdff717e1ba0aba4f6#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9faas%E3%82%A4%E3%83%99%E3%83%B3%E3%83%88pull%E5%9E%8B)

[戻る](step1.md) | [次へ](step3.md)
