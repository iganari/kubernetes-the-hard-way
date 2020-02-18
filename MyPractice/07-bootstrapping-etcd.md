# 07. Bootstrapping the etcd Cluster

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

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

以降は controller instance にログイン後の作業です ---> :police_car:

## 2. Download and Install the etcd Binaries

+ :police_car: ユーザ確認

```
whoami
```
```
### 例

root@controller-0:~# whoami
root
```

+ :police_car: [etcd](https://github.com/etcd-io/etcd) のバイナリをダウンロードし、インストールしていきます
  + 最新バージョンは [v3.4.3](https://github.com/etcd-io/etcd/releases/tag/v3.4.3) なので、こちらを使います

```
_version='v3.4.3'

wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/${_version}/etcd-${_version}-linux-amd64.tar.gz"
```

+ gzip ファイルを伸長し、PATH の通っているディレクトリに移動します

```
tar -xvf etcd-${_version}-linux-amd64.tar.gz
sudo mv etcd-${_version}-linux-amd64/etcd* /usr/local/bin/
```


+ hoge

```

```


+ hoge

```

```


+ hoge

```

```


+ hoge

```

```


## 次のステップへ :rocket:

ここまでで、 07. Bootstrapping the etcd Cluster が完了です :raised_hands:

次は [08. Bootstrapping the Kubernetes Control Plane](./08-bootstrapping-kubernetes-controllers.md) に進みます!! :muscle:
