# 05. Generating Kubernetes Configuration Files for Authentication

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ Client Authentication Configs
  + この章では、 client と admin ユーザ用の `controller manager` `kubelet` `kube-proxy` `scheduler` の kubeconfig を作成します。 　
+ Generating Kubernetes Configuration Files for Authentication
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/05-kubernetes-configuration-files.md

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

## 1. Kubernetes Public IP Address

WIP

## 2. The kubelet Kubernetes Configuration File

WIP

## 3. The kube-proxy Kubernetes Configuration File

WIP

## 4. The kube-controller-manager Kubernetes Configuration File

WIP

## 5. The kube-scheduler Kubernetes Configuration File

WIP

## 6. The admin Kubernetes Configuration File

WIP

## 7. Distribute the Kubernetes Configuration Files

WIP




























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

ここまでで、 04. Provisioning a CA and Generating TLS Certificates が完了です :raised_hands:

次は [WIP]() に進みます!! :muscle:
