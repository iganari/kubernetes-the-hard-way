# 02. Installing the Client Tools

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ gcloud コマンド以外のコマンドの準備します。
  + cfssl, cfssljson, and kubectl.
  + [Prepare](./00_prepare.md) で作成した VM は Ubuntu を使用しているので、 Linux 用のインストール方法を使用します。
+ Installing the Client Tools
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md

## 作業

<details>
<summary>(不要な場合はスキップ) GCE を起動します。</summary>

## GCE を起動します。

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

</details>

## :computer: CFSSL コマンドを VM にインストールします。

+ CFSSL コマンドとは
  + CloudFlare が公開している PKI/TLS ツールキットです。
+ GitHub
  + https://github.com/cloudflare/cfssl
+ ブログ
  + https://blog.cloudflare.com/introducing-cfssl/

Linux 用のインストールを行っていきます。

+ :package: バイナリをインターネット上からダウロードします。

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/linux/cfssljson
```

+ :package: `cfssl` `cfssljson` に実行権限を付与します。

```
chmod +x cfssl cfssljson
```

+ :package: cfssl が実行出来るか確認をします。

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

+ :package: cfssljson が実行出来るか確認をします。

```
cfssljson --version
```
```
### 例

$ cfssljson --version
Version: 1.3.4
Revision: dev
Runtime: go1.13
```

## kubectl コマンドを VM にインストールします。

+ kubectl コマンドとは
  + Kubernetes API サーバにコマンドを実行する CLI ツールです。

+ :package: バイナリをインターネット上からダウロードします。
  + 最新版のダウンロード方法 : [Install kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```

+ :package: `kubectl` に実行権限を付与します。

```
chmod +x kubectl
```

+ :package: PATH が通っているディレクトリに移動させます。

```
sudo mv kubectl /usr/local/bin/
```

+ :package: kubectl が実行出来るか確認をします。

```
kubectl version
OR
kubectl version --client
```
```
### 例

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

<details>
<summary>(不要な場合はスキップ) 作業が終わったら、 GCE は停止しておきましょう。。</summary>

## 作業が終わったら、 GCE は停止しておきましょう。

+ :package: GCE から SSH ログアウト

```
exit
```

+ :computer: GCE の停止コマンド

```
gcloud beta compute instances stop kubernetes-the-hard-way-vm
```

</details>

## 次のステップへ :rocket:

ここまでで、 02. Installing the Client Tools が完了です :raised_hands:

次は [03. Provisioning Compute Resources](./03-provisioning-compute-resources.md) に進みます!! :muscle:
