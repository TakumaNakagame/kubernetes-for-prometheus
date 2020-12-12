# kubernetes-for-prometheus

## Setup
本リポジトリを利用するために、予めCloneしておきましょう。

```
$ git clone https://github.com/TakumaNakagame/kubernetes-for-prometheus.git
```

### kubernetes-mixinのビルド
公式の[README.md](https://github.com/kubernetes-monitoring/kubernetes-mixin#generate-config-files)より以下の通り実行。
なお、環境によっては`jb`が`command not found`となるので、`~/go/bin/jb`などに置き換えてください。
```
$ go get github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb
$ brew install jsonnet

$ git clone https://github.com/kubernetes-monitoring/kubernetes-mixin
$ cd kubernetes-mixin
$ jb install

$ make prometheus_alerts.yaml
$ make prometheus_rules.yaml
```
`prometheus_alerts.yml`と`prometheus_rules.yml`が生成されました。  
Kubernetes上で利用するために、ConfigMapを生成します。

```
$ kubectl create configmap prometheus-alerts --from-file=prometheus_alerts.yml -o yaml --dry-run=client > prometheus-alerts.yaml
$ kubectl create configmap prometheus-rules --from-file=prometheus_rules.yml -o yaml --dry-run=client > prometheus-rules.yaml
```

### クラスタの作成
Kindを利用したクラスタを作成します。

対象イメージは`1.18.8`を利用していますが、必要に応じて書き換えてください。
```
$ kind create cluster --config cluster.yml --name prom-k8s
```

### Prometheusのデプロイ
リポジトリにもともとあった`prometheus.yaml`と、ビルドした`prometheus-alerts.yaml`と`prometheus-rules.yaml`をデプロイします。
`prometheus.yaml`には、最小構成の`Deployment, Service, ConfigMap`が構成されています。
なお、Prometheusは`--web.enable.lifecycle`を有効にしているため、`curl -X POST http://localhost:9090/-/reload`によるホットリロードを可能にしています。

```
$ kubectl apply -f prometheus.yaml
$ kubectl apply -f prometheus-alerts.yaml
$ kubectl apply -f prometheus-rules.yaml
```
