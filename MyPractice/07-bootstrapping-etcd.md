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
  + :warning:
  + gcloud compute ssh は実行するユーザを対象の VM 上にも作成します。
  + root ユーザ以外で実行し、VM 上でも root ユーザ以外の sudo 実行出来るユーザで進めていく認識で見てください。

```
gcloud compute ssh controller-0
```

+ なお、このような作業の際は tmux などのツールが有用です

以降は controller instance にログイン後の作業です ---> :police_car: `controller-0` `controller-1` `controller-2`

## 2. Download and Install the etcd Binaries

+ :police_car: ユーザ確認

```
whoami
```
```
### 例

iganari@controller-0:~$ whoami
iganari
```

+ :police_car: [etcd](https://github.com/etcd-io/etcd) のバイナリをダウンロードし、インストールしていきます
  + 最新バージョンは [v3.4.3](https://github.com/etcd-io/etcd/releases/tag/v3.4.3) なので、こちらを使います

```
_version='v3.4.3'

wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/${_version}/etcd-${_version}-linux-amd64.tar.gz"
```

+ :police_car: gzip ファイルを伸長し、PATH の通っているディレクトリに移動します

```
tar -xvf etcd-${_version}-linux-amd64.tar.gz
sudo mv etcd-${_version}-linux-amd64/etcd* /usr/local/bin/
```

## 3. Configure the etcd Server

+ :police_car: 事前に作成したバイナリファイルを移動します。

```
sudo mkdir -p /etc/etcd /var/lib/etcd
sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
```


+ :police_car: 現在の VM の内部 IP アドレスを取得します。

```
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
```
```
### 例

iganari@controller-0:~$ echo ${INTERNAL_IP}
10.240.0.10
```

+ :police_car: etcs クラスタの内で一意の名前を持っている必要があり、今回は GCP 上の名前と同一のものを設定しておきます。

```
ETCD_NAME=$(hostname -s)
```
```
### 例

iganari@controller-0:~$ echo ${ETCD_NAME}
controller-0
```

+ :police_car: ここで一度、すべての Control Plane 用の VM の内部 IP アドレスを集めておきます
  + それぞれの VM 内で調べてもいいですし、CLIでもGUIでも確認できます。

```
INTERNAL_IP_0='10.240.0.10'
INTERNAL_IP_1='10.240.0.11'
INTERNAL_IP_2='10.240.0.12'
```

+ etcd で使用する etcd.service を作りましょう

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://${INTERNAL_IP_0}:2380,controller-1=https://${INTERNAL_IP_1}:2380,controller-2=https://${INTERNAL_IP_2}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
```
### 確認

iganari@controller-0:~$ cat /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \
  --name controller-0 \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://10.240.0.10:2380 \
  --listen-peer-urls https://10.240.0.10:2380 \
  --listen-client-urls https://10.240.0.10:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.240.0.10:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---> 上記の作業を `controller-0` `controller-1` `controller-2` のすべてで行いましょう



## 4. Start the etcd Server

+ etcd server を起動しておきましょう
  + `controller-0` `controller-1` `controller-2` のすべての VM で実行する必要があります

```
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
```

## 5. Verification

+ etcd のクラスタの確認を行います。

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```

+ controller-0 にて確認

```
iganari@controller-0:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
```

+ controller-1 にて確認

```
iganari@controller-1:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
```

+ controller-2 にて確認

```
iganari@controller-2:~$ sudo ETCDCTL_API=3 etcdctl member list \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem
3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false
```

---> すべて同じ出力結果が得られました!!

## 次のステップへ :rocket:

ここまでで、 07. Bootstrapping the etcd Cluster が完了です :raised_hands:

次は [08. Bootstrapping the Kubernetes Control Plane](./08-bootstrapping-kubernetes-controllers.md) に進みます!! :muscle:
