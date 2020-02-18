# 07. Bootstrapping the etcd Cluster

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ Bootstrapping the etcd Cluster
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/07-bootstrapping-etcd.md
+ 詳細
  + WIP

## 1. 注意

+ この章では、 各 controller instance にて作業をします
  + `controller-0` , `controller-2` , `controller-2`
  + したがって、以下の作業は gcloud コマンドなどで、各インスタンスにログイン後に行ってください
+ gcloud を用いた SSH ログインをするコマンド例

```
gcloud compute ssh controller-0
```

+ なお、このような作業の際は tmux などのツールが有用です

以降は controller instance にログイン後の作業です

## 2. Download and Install the etcd Binaries

+ ユーザ確認

```
whoami
```
```
### 例

root@controller-0:~# whoami
root
```



## 次のステップへ :rocket:

ここまでで、 07. Bootstrapping the etcd Cluster が完了です :raised_hands:

次は [08. Bootstrapping the Kubernetes Control Plane](./08-bootstrapping-kubernetes-controllers.md) に進みます!! :muscle:
