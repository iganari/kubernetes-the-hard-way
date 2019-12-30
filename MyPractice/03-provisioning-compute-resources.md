# 03. Provisioning Compute Resources

## 注意点

+ アイコンの説明
  + :computer:
    + ホストマシン
  + :package:
    + GCE

## このページで参考にする本家のページ

+ Provisioning Compute Resources
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md

## やること

+ WIP
  + K8s は K8s のコントロールプレーンのマシンと、コンテナを実行するワーカーノードが必要になります。
  + セキュアで高可用性の Kubernetes クラスタを 1つの Zone 内 で作成ます


## GCE を起動します

[Prepare](./00_prepare.md) で作成した VM を gcloud コマンドで起動します。

+ :computer: GCP と認証を通します。

```
gcloud auth login
```

+ :computer: gcloud コマンドの設定を行います。

```
gcloud config set project iganari-k8s-hardway-pre
gcloud config set compute/zone asia-northeast1-c
```

+ :computer: VM の起動を行います。

```
gcloud beta compute instances start kubernetes-the-hard-way-vm
```

+ :computer: この GCE に SSH して作業を行います。
  + ここで、 :package: での作業に切り替わります。

```
gcloud compute ssh kubernetes-the-hard-way-vm
```

+ :package: 作業ユーザ(iganari)を変更します。

```
su - iganari
```

## ネットワークを作成する

### VPC ネットワーク

+ `kubernetes-the-hard-way` という名前の VPC ネットワークを作成します。
  + 余計なサブネットを作成しないように、 `--subnet-mode custom` オプションを使用します。

```
gcloud compute networks create kubernetes-the-hard-way \
  --subnet-mode custom
```

### サブネット

+ 上記で作成した VPC ネットワーク内にサブネットを作成します。
  + プライベート IP アドレスは `--range` にて指定します。
  + サブネットを作成するリージョンは `--region` で指定します。

```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24 \
  --region asia-northeast1
```

### Firewall Rule

+ 内部ネットワーク用の Firewall Rule を作成します。
  + 対象プロトコルは、 `TCP` , `UDP` , `ICMP`
  + 適用範囲は VPC network の `kubernetes-the-hard-way` のみ
  + 送信元として許可する IP アドレスの範囲は、 `10.240.0.0/24` , `10.200.0.0/16`

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

+ 外部からの通信を許可する Firewall Rule を作成します。
  + 対象プロトコルとポートは、 `TCPの 22番ポート` , `TCP の6443番ポート` , `ICMP`
    + 22 番ポートは SSH ログイン用
    + 6443 番ポートは Kubernetes の API サーバ 用 [Controlling Access to the Kubernetes API](https://kubernetes.io/docs/reference/access-authn-authz/controlling-access/)
    + 元のドキュメントにも書いてあります
      + `An external load balancer will be used to expose the Kubernetes API Servers to remote clients.`

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-external \
  --allow tcp:22,tcp:6443,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 0.0.0.0/0
```

+ 作成した Firewall Rule を確認します。

```
gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
```
```
### 例

$ gcloud compute firewall-rules list --filter="network:kubernetes-the-hard-way"
NAME                                    NETWORK                  DIRECTION  PRIORITY  ALLOW                 DENY  DISABLED
kubernetes-the-hard-way-allow-external  kubernetes-the-hard-way  INGRESS    1000      tcp:22,tcp:6443,icmp        False
kubernetes-the-hard-way-allow-internal  kubernetes-the-hard-way  INGRESS    1000      tcp,udp,icmp                False
```

### 静的外部 IP アドレスを予約

+ VPC network の `デフォルト` 内に、静的外部 IP アドレスを予約します。
  + VM にアタッチしないと :moneybag: 課金が発生します







