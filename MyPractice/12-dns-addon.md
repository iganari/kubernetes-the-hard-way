# 12. Deploying the DNS Cluster Add-on

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Deploying the DNS Cluster Add-on
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/12-dns-addon.md
+ 詳細
  + WIP

## 1. 注意

+ この章では、 各 worker instance にて作業をします
  + `controller-0` , `controller-2` , `controller-2`
  + したがって、以下の作業は gcloud コマンドなどで、各インスタンスにログイン後に行ってください
+ gcloud を用いた SSH ログインをするコマンド例
  + :warning:
  + gcloud compute ssh は実行するユーザを対象の VM 上にも作成します。
  + root ユーザ以外で実行し、VM 上でも root ユーザ以外の sudo 実行出来るユーザで進めていく認識で見てください。

```
gcloud compute ssh controller-0
```

+ なお、このような作業の際は tmux などのツールが有用です

以降は controller instance にログイン後の作業です ---> :police_car: `controller-0` `controller-1` `controller-2`

## 2. The DNS Cluster Add-on

+ WIP

```

```

## 3. WIP

+ WIP

```

```

## 4. WIP

+ WIP

```

```

## 5. WIP

+ WIP

```

```

## 次のステップへ :rocket:

ここまでで、12. Deploying the DNS Cluster Add-on が完了です :raised_hands:

次は [13. Smoke Test](./13-smoke-test.md) に進みます!! :muscle:
