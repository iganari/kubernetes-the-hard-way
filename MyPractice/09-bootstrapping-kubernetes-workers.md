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

+ :car: `containerd` の設定ファイルを作成します

```
sudo mkdir -p /etc/containerd/
```
```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF
```

+ :car: systemctl 用の設定ファイル `containerd.service` を作成します

```
cat <<EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF
```

### 2-6. Configure the Kubelet

+ :car: 事前に作成した設定ファイル及び pem ファイルを移動します

```
sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/
```
```
### 確認

iganari@worker-0:~$ ls ${HOSTNAME}-key.pem
worker-0-key.pem
iganari@worker-0:~$ ls ${HOSTNAME}.kubeconfig
worker-0.kubeconfig
iganari@worker-0:~$ ls ca.pem
ca.pem
```

+ :car: systemctl 用の設定ファイル `kubelet-config.yaml` を作成します

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

+ :car: systemctl 用の設定ファイル `kubelet.service` を作成します

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```


### 2-7. Configure the Kubernetes Proxy

+ :car: 必要なファイルの確認

```
ls kube-proxy.kubeconfig
```
```
### 例

iganari@worker-0:~$ ls kube-proxy.kubeconfig
kube-proxy.kubeconfig
```

+ :car: 事前に作成した設定ファイルを移動します

```
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

+ :car: systemctl 用の設定ファイル `kube-proxy-config.yaml` を作成します

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

+ :car: systemctl 用の設定ファイル `kube-proxy.service` を作成します 

```
cat <<EOF | sudo tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```




### 2-8. Start the Worker Services

+ :car: Worker Services を起動しましょう

```
sudo systemctl daemon-reload
sudo systemctl enable containerd kubelet kube-proxy
sudo systemctl start containerd kubelet kube-proxy
```
```
### 例

$ sudo systemctl status containerd kubelet kube-proxy
● containerd.service - containerd container runtime
   Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-02-20 04:08:25 UTC; 31s ago
     Docs: https://containerd.io
  Process: 18429 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
 Main PID: 18437 (containerd)
    Tasks: 9 (limit: 4395)
   CGroup: /system.slice/containerd.service
           └─18437 /bin/containerd
```


## 2. Verification

+ :package: 踏み台サーバから確認します 

```
gcloud compute ssh controller-0 \
  --command "kubectl get nodes --kubeconfig admin.kubeconfig"
```
```
### 例

$ gcloud compute ssh controller-0   --command "kubectl get nodes --kubeconfig admin.kubeconfig"
NAME       STATUS   ROLES    AGE   VERSION
worker-0   Ready    <none>   38s   v1.15.3
worker-1   Ready    <none>   35s   v1.15.3
worker-2   Ready    <none>   38s   v1.15.3
```

---> これで Worker Nodes の作成が完了しました!!

## 次のステップへ :rocket:

ここまでで、 09. Bootstrapping the Kubernetes Worker Nodes が完了です :raised_hands:

次は [WIP]() に進みます!! :muscle:
