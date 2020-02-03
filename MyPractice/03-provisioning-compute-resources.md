# 03. Provisioning Compute Resources

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ WIP
  + K8s は K8s のコントロールプレーンのマシンと、コンテナを実行するワーカーノードが必要になります。
  + セキュアで高可用性の Kubernetes クラスタを 1つの Zone 内 で作成ます
+ Provisioning Compute Resources
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md

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


## 1. ネットワークを作成する

### VPC ネットワーク

+ :package: `kubernetes-the-hard-way` という名前の VPC ネットワークを作成します。
  + 余計なサブネットを作成しないように、 `--subnet-mode custom` オプションを使用します。

```
gcloud compute networks create kubernetes-the-hard-way \
  --subnet-mode custom
```

### サブネット

+ :package: 上記で作成した VPC ネットワーク内にサブネットを作成します。
  + プライベート IP アドレスは `--range` にて指定します。
  + サブネットを作成するリージョンは `--region` で指定します。

```
gcloud compute networks subnets create kubernetes \
  --network kubernetes-the-hard-way \
  --range 10.240.0.0/24 \
  --region asia-northeast1
```

### Firewall Rule

+ :package: 内部ネットワーク用の Firewall Rule を作成します。
  + 対象プロトコルは、 `TCP` , `UDP` , `ICMP`
  + 適用範囲は VPC network の `kubernetes-the-hard-way` のみ
  + 送信元として許可する IP アドレスの範囲は、 `10.240.0.0/24` , `10.200.0.0/16`

```
gcloud compute firewall-rules create kubernetes-the-hard-way-allow-internal \
  --allow tcp,udp,icmp \
  --network kubernetes-the-hard-way \
  --source-ranges 10.240.0.0/24,10.200.0.0/16
```

+ :package: 外部からの通信を許可する Firewall Rule を作成します。
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

+ :package: 作成した Firewall Rule を確認します。

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

+ :package: VPC network の `デフォルト` 内に、静的外部 IP アドレスを予約します。
  + ~~:moneybag: VM にアタッチしないと課金が発生します~~
  + :moneybag: 2020年1月より、VM にアタッチしていても有料の対象となりました。

```
### WIP
gcloud compute addresses create kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region)
```

+ :package: 確認コマンド

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```
```
### 例

WIP
```

## 2. GCE を構築

### Kubernetes の コントロールプレーンになるインスタンスを作成します。

+ :package: 3台作成するために Bash の for 文を用いて、一気に作成します。

```
for i in 0 1 2; do
  gcloud compute instances create controller-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --private-network-ip 10.240.0.1${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,controller
done
```


### Kubernetes の worker node になるインスタンスを作成

+ :package: 3台作成するために Bash の for 文を用いて、一気に作成します。
  + metadata に注意

```
for i in 0 1 2; do
  gcloud compute instances create worker-${i} \
    --async \
    --boot-disk-size 200GB \
    --can-ip-forward \
    --image-family ubuntu-1804-lts \
    --image-project ubuntu-os-cloud \
    --machine-type n1-standard-1 \
    --metadata pod-cidr=10.200.${i}.0/24 \
    --private-network-ip 10.240.0.2${i} \
    --scopes compute-rw,storage-ro,service-management,service-control,logging-write,monitoring \
    --subnet kubernetes \
    --tags kubernetes-the-hard-way,worker
done
```

### 確認

+ :package: 作成したインスタンスを確認します
  + 正常に出来ていれば、 controller-0 ~ controller-2, worker-0 ~ worker-2 が出来ています。

```
gcloud compute instances list
```
```
### 例

WIP
```

## 3. SSH ログイン

+ :package: 作成したインスタンスに gcloud 経由で SSH ログインしてみます。

```
### 例

gcloud compute ssh controller-0
```

+ :package: 無事に SSH ログインが出来たら、確認は以上です。

```
exit
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

ここまでで、 03. Provisioning Compute Resources が完了です :raised_hands:

次は [04. Provisioning a CA and Generating TLS Certificates](./04-certificate-authority.md) に進みます!! :muscle:
