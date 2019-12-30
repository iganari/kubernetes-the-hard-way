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
