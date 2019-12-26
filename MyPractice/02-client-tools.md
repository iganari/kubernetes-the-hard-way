# 02. Installing the Client Tools

## 注意点

+ アイコンの説明
  + :computer:
    + ホストマシン
  + :package:
    + GCE

## このページで参考にする本家のページ

+ Installing the Client Tools
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md

## やること

+ :package: gcloud コマンド以外のコマンドの準備します。
  + cfssl, cfssljson, and kubectl.
  + [Prepare](./00_prepare.md) で作成した VM は Ubuntu を使用しているので、 Linux 用のインストール方法を使用します。

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

+ :package: 

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson
```

+ :package: `cfssl` `cfssljson` に実行権限を付与します。

```
chmod +x cfssl cfssljson
```

+ :package: PATH が通っているディレクトリに移動させます。

```
sudo mv cfssl cfssljson /usr/local/bin/

```

+ 実行出来るか確認をします。

```
cfssl version
```
```
### 例

$ cfssl version
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

## :computer: Install CFSSL

+ CFSSL コマンドとは
  + CloudFlare が公開している PKI/TLS ツールキットです。
+ GitHub
  + https://github.com/cloudflare/cfssl
+ ブログ
  + https://blog.cloudflare.com/introducing-cfssl/

Linux 用のインストールを行っていきます。


## Install kubectl

WIP
