# 04. Provisioning a CA and Generating TLS Certificates


## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)

## この章でやること

+ WIP
  + cfssl コマンドを持ちいて、鍵の作成
  + 作成した鍵を VM に転送
+ Provisioning a CA and Generating TLS Certificates
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md

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

## 1. 鍵作成 ([Certificate Authority](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md#certificate-authority))

### `CA configuration file (CA)` , `certificate` , and `private key` を作成します。

+ CA の作成

```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
```

+ CA 証明書サインファイルの作成 

```
cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
```

+ `cfssl` コマンドを用いて、 CA 証明書とキーファイルを作成します。

```
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
```
### 例

$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2020/02/09 23:53:52 [INFO] generating a new CA key and certificate from CSR
2020/02/09 23:53:52 [INFO] generate received request
2020/02/09 23:53:52 [INFO] received CSR
2020/02/09 23:53:52 [INFO] generating key: rsa-2048
2020/02/09 23:53:52 [INFO] encoded CSR
2020/02/09 23:53:52 [INFO] signed certificate with serial number 131245954278832858228666587662728120599709170889
$
$ ls -1
ca-config.json
ca-csr.json
ca-key.pem
ca.csr
ca.pem
```

### `admin` 用の 証明書とキーファイルを作成します。

```
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:masters",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
```

+ `cfssl` コマンドを用いて、 CA 証明書とキーファイルを作成します。
  + 実行すると waring が出るが、ここでは気にしない…

```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
```
```
$ ls -1
admin-csr.json
admin-key.pem
admin.csr
admin.pem
ca-config.json
ca-csr.json
ca-key.pem
ca.csr
ca.pem
```

### Kubernetes の worker node 用の CA 証明書とキーファイルを作成します。

公式のやり方よりもわかりやすく分解しました

+ worker-0 用

```
export _node_name='worker-0'

cat > ${_node_name}-csr.json <<EOF
{
  "CN": "system:node:${_node_name}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
EXTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].networkIP)')
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${_node_name}-csr.json | cfssljson -bare ${_node_name}
```

+ worker-1 用

```
export _node_name='worker-1'

cat > ${_node_name}-csr.json <<EOF
{
  "CN": "system:node:${_node_name}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
EXTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].networkIP)')
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${_node_name}-csr.json | cfssljson -bare ${_node_name}
```

+ worker-2 用

```
export _node_name='worker-2'

cat > ${_node_name}-csr.json <<EOF
{
  "CN": "system:node:${_node_name}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:nodes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
EXTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].accessConfigs[0].natIP)')

INTERNAL_IP=$(gcloud compute instances describe ${_node_name} \
  --format 'value(networkInterfaces[0].networkIP)')
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -hostname=${instance},${EXTERNAL_IP},${INTERNAL_IP} \
  -profile=kubernetes \
  ${_node_name}-csr.json | cfssljson -bare ${_node_name}
```

+ 現状のローカル

```
$ ls -1
admin-csr.json
admin-key.pem
admin.csr
admin.pem
ca-config.json
ca-csr.json
ca-key.pem
ca.csr
ca.pem
worker-0-csr.json
worker-0-key.pem
worker-0.csr
worker-0.pem
worker-1-csr.json
worker-1-key.pem
worker-1.csr
worker-1.pem
worker-2-csr.json
worker-2-key.pem
worker-2.csr
worker-2.pem
```

### kube-controller-manager 用の CA 証明書とキーファイルを作成します。

```
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-controller-manager",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

+ 確認

```
$ ls -1 | grep kube-controller-manager
kube-controller-manager-csr.json
kube-controller-manager-key.pem
kube-controller-manager.csr
kube-controller-manager.pem
```

### kube-proxy 用の CA 証明書とキーファイルを作成します。

```
cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:node-proxier",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy
```

+ 確認

```
$ ls -1 | grep kube-proxy
kube-proxy-csr.json
kube-proxy-key.pem
kube-proxy.csr
kube-proxy.pem
```

### kube-scheduler 用の CA 証明書とキーファイルを作成します。

```
cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "system:kube-scheduler",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
$ ls -1 | grep kube-scheduler
kube-scheduler-csr.json
kube-scheduler-key.pem
kube-scheduler.csr
kube-scheduler.pem
```

### API Server 用の CA 証明書とキーファイルを作成します。

+ 静的 IP アドレスの確認

```
gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
```
```
### 例

$ gcloud compute addresses list --filter="name=('kubernetes-the-hard-way')"
NAME                     ADDRESS/RANGE  TYPE      PURPOSE  NETWORK  REGION           SUBNET  STATUS
kubernetes-the-hard-way  34.84.1.43     EXTERNAL                    asia-northeast1          RESERVED
```

+ (上記の結果を元に) region の設定し、変数を入れる

```
gcloud config set compute/region asia-northeast1
```
```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')

KUBERNETES_HOSTNAMES=kubernetes,kubernetes.default,kubernetes.default.svc,kubernetes.default.svc.cluster,kubernetes.svc.cluster.local
```
```
cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```

+ 確認します

```
$ ls -1 | grep kubernetes
kubernetes-csr.json
kubernetes-key.pem
kubernetes.csr
kubernetes.pem
```

### service-account 用の CA 証明書とキーファイルを作成します。

```
cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "Kubernetes The Hard Way",
      "ST": "Oregon"
    }
  ]
}
EOF
```
```
cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
```
```
$ ls -1 | grep service-account
service-account-csr.json
service-account-key.pem
service-account.csr
service-account.pem
```
