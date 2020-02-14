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

+ 先に作成した `External IP addresses` を確認します。

```
KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
  --region $(gcloud config get-value compute/region) \
  --format 'value(address)')
```
```
### 例

$ KUBERNETES_PUBLIC_ADDRESS=$(gcloud compute addresses describe kubernetes-the-hard-way \
>   --region $(gcloud config get-value compute/region) \
>   --format 'value(address)')
Your active configuration is: [kubernetes-the-hard-way]
$
$ echo $KUBERNETES_PUBLIC_ADDRESS
34.84.1.43
```

## 2. The kubelet Kubernetes Configuration File

+ 各 Worker Node に対して、 kubeconfig を作っていきます

```
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

+ 確認します。

```
$ ls | grep .kubeconfig
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig
```

## 3. The kube-proxy Kubernetes Configuration File

+ `kube-proxy` のための、kubeconfig を作成します。

```
vim kube-proxy.kubeconfig
```
```
### 内容

{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```


## 4. The kube-controller-manager Kubernetes Configuration File

+ `kube-controller-manager` のための、kubeconfig を作成します。

```
vim kube-controller-manager.kubeconfig
```
```
### 内容


{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```



## 5. The kube-scheduler Kubernetes Configuration File

+ `kube-scheduler` のための、kubeconfig を作成します。

```
vim kube-scheduler.kubeconfig
```
```
### 内容


{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```

## 6. The admin Kubernetes Configuration File

+ `admin(ユーザ)` のための、kubeconfig を作成します。

```
vim admin.kubeconfig
```
```
### 内容


{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```

## 7. Distribute the Kubernetes Configuration Files

+ 各 Worker Node に対して、 `kubelet` と `kube-proxy` の設定をコピーします

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done
```

+ 各 Controller Node に対して、 `kube-controller-manager` と `kube-scheduler` の設定をコピーします


```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
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

ここまでで、 04. Provisioning a CA and Generating TLS Certificates が完了です :raised_hands:

次は [WIP]() に進みます!! :muscle:
