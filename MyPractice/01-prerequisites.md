# 01. prerequisites

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ gcloud コマンドの簡単な設定を行います。
+ このページで参考にする本家のページ
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/01-prerequisites.md

## 作業

+ :package: gcloud コマンドの準備
  + Google Cloud SDK のバージョンが 262.0.0 より高いことを確認します。

```
gcloud --version
```
```
### 例

$ gcloud --version
Google Cloud SDK 274.0.0
alpha 2019.12.17
beta 2019.12.17
bq 2.0.51
core 2019.12.17
gsutil 4.46
kubectl 2019.12.17
```

+ :package: gcloud コマンドの設定を先に作成します。
  + 本家では `us-west1-c` をデフォルトにしてますが、東京リージョン `asia-northeast1` に変更します。

```
gcloud auth application-default login
```
```
gcloud config set compute/region asia-northeast1
gcloud config set compute/zone asia-northeast1-c
```

+ :package: gcloud の設定を確認します。

```
gcloud config configurations list
```
```
### 例

$ gcloud config configurations list
NAME                            IS_ACTIVE  ACCOUNT  PROJECT                          DEFAULT_ZONE       DEFAULT_REGION
kubernetes-the-hard-way-vm      True                kubernetes-the-hard-way-pre      asia-northeast1-c  asia-northeast1
```

## tmux のすゝめ

このチュートリアルでは複数のコンピュートインスタンスに対して、同じコマンドを実行するため、 tmux の synchronize-panes のような機能があったほうがいいよとのことです。

## 次のステップへ :rocket:

ここまでで、 01. prerequisites が完了です :raised_hands:

次は [Installing the Client Tools: クライアントツールのインストール](02-client-tools.md) に進みます!! :muscle:
