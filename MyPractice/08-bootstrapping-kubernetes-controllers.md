# 08. Bootstrapping the Kubernetes Control Plane

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Bootstrapping the Kubernetes Control Plane
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/08-bootstrapping-kubernetes-controllers.md
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

## 2. Provision the Kubernetes Control Plane

### 2-1. Download and Install the Kubernetes Controller Binaries

+ :police_car: Kubernetes の設定ファイルを設置するディレクトリを作成します

```
sudo mkdir -p /etc/kubernetes/config
```

+ :police_car: Kubernetes Controller のバイナリをダウンロードします

```
wget -q --show-progress --https-only --timestamping \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-apiserver" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-controller-manager" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kube-scheduler" \
  "https://storage.googleapis.com/kubernetes-release/release/v1.15.3/bin/linux/amd64/kubectl"
```
+ :police_car: ダウンロードしたバイナリを PATH の通ったディレクトリに移動します

```
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
```


### 2-2. Configure the Kubernetes API Server

+ :police_car: Kubernetes API Server 用のディレクトリを作成し、事前に作成した pem と YAML を移動します

```
sudo mkdir -p /var/lib/kubernetes/
```
```
sudo mv ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
  service-account-key.pem service-account.pem \
  encryption-config.yaml /var/lib/kubernetes/
```

+ :police_car: 確認します

```
ls /var/lib/kubernetes/
```
```
### 例

iganari@controller-0:~$ ls /var/lib/kubernetes/
ca-key.pem  ca.pem  encryption-config.yaml  kubernetes-key.pem  kubernetes.pem  service-account-key.pem  service-account.pem
```

+ :police_car: 内部 IP アドレスの取得

```
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)
```

+ :police_car: 確認

```
echo ${INTERNAL_IP}
```

+ :police_car: `kube-apiserver.service` 用の systemd の設定ファイルを作成します

```
INTERNAL_IP_0='10.240.0.10'
INTERNAL_IP_1='10.240.0.11'
INTERNAL_IP_2='10.240.0.12'
```
```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://${INTERNAL_IP_0}:2379,https://${INTERNAL_IP_1}:2379,https://${INTERNAL_IP_2}:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config=api/all \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

+ :police_car: 確認

```
cat /etc/systemd/system/kube-apiserver.service
```

### 2-3. Configure the Kubernetes Controller Manager

+ :police_car: `kube-controller-manager` 用の設定ファイルを設置するディレクトリを作成します

```
sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/
```

+ :police_car: `kube-controller-manager` 用の systemd の設定ファイルを作成します
  + 本家に `--master=127.0.0.1:8080` を追加しています

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2 \
  --master=127.0.0.1:8080
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

+ :police_car: 確認

```
cat /etc/systemd/system/kube-controller-manager.service
```

### 2-4. Configure the Kubernetes Scheduler


+ :police_car: `kube-scheduler` 用の設定ファイルを設置するディレクトリを作成します

```
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
```

+ :police_car: `kube-scheduler` 用の設定ファイルを作成します

```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

+ :police_car: `kube-scheduler` 用の systemd の設定ファイルを作成します
  + 本家に `--master=127.0.0.1:8080` を追加しています

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2 \
  --master=127.0.0.1:8080
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 2-5. Start the Controller Services

+ :police_car: ここまで作成してきた Service を起動します

```
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```

### 2-6. Enable HTTP Health Checks

```
sudo apt-get update
sudo apt-get install -y nginx
```
```
cat > kubernetes.default.svc.cluster.local <<EOF
server {
  listen      80;
  server_name kubernetes.default.svc.cluster.local;

  location /healthz {
     proxy_pass                    https://127.0.0.1:6443/healthz;
     proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
  }
}
EOF
```

+ :police_car: nignx の設定ファイルを移動します

```
sudo mv kubernetes.default.svc.cluster.local \
  /etc/nginx/sites-available/kubernetes.default.svc.cluster.local

sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local \
  /etc/nginx/sites-enabled/
```

+ :police_car: nignx の設定を反映させるための再起動と自動起動を設定します。

```
sudo systemctl restart nginx
sudo systemctl enable nginx
```

+ :police_car: 確認

```
sudo systemctl status nginx
```

### 2-7. Verification

+ :police_car: 確認

```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
```
### 例

iganari@controller-0:~$ kubectl get componentstatuses --kubeconfig admin.kubeconfig
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-2               Healthy   {"health":"true"}
etcd-1               Healthy   {"health":"true"}
etcd-0               Healthy   {"health":"true"}
```

+ :police_car: 確認

```
curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
```
```
iganari@controller-0:~$ curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Wed, 19 Feb 2020 15:45:00 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 2
Connection: keep-alive
X-Content-Type-Options: nosniff

```

## 3. RBAC for Kubelet Authorization























## 次のステップへ :rocket:

ここまでで、 08. Bootstrapping the Kubernetes Control Plane が完了です :raised_hands:

次は [09. Bootstrapping the Kubernetes Worker Nodes](./09-bootstrapping-kubernetes-workers.md) に進みます!! :muscle:
