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

## 1. Kubernetes Public IP Address

+ :package: 先に作成した `External IP addresses` を確認します。

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

+ :package: 各 Worker Node に対して、 kubeconfig を作っていきます

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

+ :package: 確認します。

```
$ ls | grep .kubeconfig
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig
```

## 3. The kube-proxy Kubernetes Configuration File

+ :package: `kube-proxy` のための、kubeconfig を作成します。

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

+ :package: `kube-controller-manager` のための、kubeconfig を作成します。

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

+ :package: `kube-scheduler` のための、kubeconfig を作成します。

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

+ :package: `admin(ユーザ)` のための、kubeconfig を作成します。

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

+ :package: 各 Worker Node に対して、 `kubelet` と `kube-proxy` の設定をコピーします

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
done
```
```
$ for instance in worker-0 worker-1 worker-2; do
>   gcloud compute scp ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}:~/
> done
worker-0.kubeconfig                                                                             100% 6368     9.2MB/s   00:00
kube-proxy.kubeconfig                                                                           100%  638     1.6MB/s   00:00
worker-1.kubeconfig                                                                             100% 6372     5.6MB/s   00:00
kube-proxy.kubeconfig                                                                           100%  638     1.2MB/s   00:00
worker-2.kubeconfig                                                                             100% 6368     4.9MB/s   00:00
kube-proxy.kubeconfig                                                                           100%  638     1.2MB/s   00:00
```


+ :package: 各 Controller Node に対して、 `kube-controller-manager` と `kube-scheduler` の設定をコピーします

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
done
```
```
$ for instance in controller-0 controller-1 controller-2; do
>   gcloud compute scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}:~/
> done
admin.kubeconfig                                                                                100%  565    27.9KB/s   00:00
kube-controller-manager.kubeconfig                                                              100%  723   123.2KB/s   00:00
kube-scheduler.kubeconfig                                                                       100%  651   348.7KB/s   00:00
admin.kubeconfig                                                                                100%  565   829.1KB/s   00:00
kube-controller-manager.kubeconfig                                                              100%  723   943.5KB/s   00:00
kube-scheduler.kubeconfig                                                                       100%  651     1.0MB/s   00:00
admin.kubeconfig                                                                                100%  565   838.8KB/s   00:00
kube-controller-manager.kubeconfig                                                              100%  723     1.2MB/s   00:00
kube-scheduler.kubeconfig                                                                       100%  651     1.3MB/s   00:00
```

## 次のステップへ :rocket:

ここまでで、 05. Generating Kubernetes Configuration Files for Authentication が完了です :raised_hands:

次は [WIP]() に進みます!! :muscle:
