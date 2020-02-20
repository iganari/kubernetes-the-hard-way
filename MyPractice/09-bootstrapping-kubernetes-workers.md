# 09. Bootstrapping the Kubernetes Worker Nodes

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Bootstrapping the Kubernetes Worker Nodes
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/09-bootstrapping-kubernetes-workers.md
+ 詳細
  + WIP

## 1. 注意

+ この章では、 各 worker instance にて作業をします
  + `worker-0` , `worker-2` , `worker-2`
  + したがって、以下の作業は gcloud コマンドなどで、各インスタンスにログイン後に行ってください
+ gcloud を用いた SSH ログインをするコマンド例
  + :warning:
  + gcloud compute ssh は実行するユーザを対象の VM 上にも作成します。
  + root ユーザ以外で実行し、VM 上でも root ユーザ以外の sudo 実行出来るユーザで進めていく認識で見てください。

```
gcloud compute ssh worker-0
```

+ なお、このような作業の際は tmux などのツールが有用です

以降は worker instance にログイン後の作業です ---> :car: `worker-0` `worker-1` `worker-2`

## 2. Provision the Kubernetes Control Plane

+ :car: 作業ユーザに変更

```
iganari@worker-0:~$ whoami
iganari
```

### 2-1. Install the OS dependencies

+ :car: 必要になるパッケージをインストールします

```
sudo apt-get update
sudo apt-get -y install socat conntrack ipset
```

### 2-2. Disable Swap

+ :car: swap を disabled にするのが推奨されている
  + デフォルトでは kubelet は swap が enabled だと失敗する
  + 下記の

+ 確認コマンド

```
sudo swapon --show
```

+ swap を disabled にする

```
sudo swapoff -a
```

### 2-3. Download and Install Worker Binaries

+ :car: バイナリをダウンロードする

```
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.15.0/crictl-v1.15.0-linux-amd64.tar.gz \
  https://github.com/opencontainers/runc/releases/download/v1.0.0-rc8/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.8.2/cni-plugins-linux-amd64-v0.8.2.tgz \
  https://github.com/containerd/containerd/releases/download/v1.2.9/containerd-1.2.9.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubelet
```

+ :car: ディレクトリの作成

```
sudo mkdir -p \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /var/lib/kubernetes \
  /var/run/kubernetes
```

+ :car: worker instance 用のバイナリをインストールする

```
mkdir containerd
tar -xvf crictl-v1.15.0-linux-amd64.tar.gz
tar -xvf containerd-1.2.9.linux-amd64.tar.gz -C containerd
sudo tar -xvf cni-plugins-linux-amd64-v0.8.2.tgz -C /opt/cni/bin/
sudo mv runc.amd64 runc
chmod +x crictl kubectl kube-proxy kubelet runc 
sudo mv crictl kubectl kube-proxy kubelet runc /usr/local/bin/
sudo mv containerd/bin/* /bin/
```

### 2-4. Configure CNI Networking

+ :car: 現在作業している VM の Pod に割り当てた CIDR の範囲を取得します

```
POD_CIDR=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/attributes/pod-cidr)
```

+ 確認

```
echo ${POD_CIDR}
```
```
### 例

iganari@worker-0:~$ echo ${POD_CIDR}
10.200.0.0/24
=====================================
iganari@worker-1:~$ echo ${POD_CIDR}
10.200.1.0/24
=====================================
iganari@worker-2:~$ echo ${POD_CIDR}
10.200.2.0/24
```

+ :car: `bridge` の設定ファイルを作成します

```
cat <<EOF | sudo tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

+ :car: `loopback` の設定ファイルを作成します

```
cat <<EOF | sudo tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
```

### 2-5. Configure containerd

+ :car: 

```
sudo swapon --show
```

### 2-6. Configure the Kubelet

+ :car: 

```
sudo swapon --show
```

### 2-7. Configure the Kubernetes Proxy

+ :car: 

```
sudo swapon --show
```

### 2-8. Start the Worker Services

+ :car: 

```
sudo swapon --show
```

## 2. Verification

+ :car: 

```
sudo swapon --show
```

## 次のステップへ :rocket:

ここまでで、 09. Bootstrapping the Kubernetes Worker Nodes が完了です :raised_hands:

次は [WIP]() に進みます!! :muscle:
