# 10. Configuring kubectl for Remote Access

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Configuring kubectl for Remote Access
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/10-configuring-kubectl.md
+ 詳細
  + admin 権限をベースとした、kebectl コマンド用の kubeconfig file を作ります

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

## 2. The Admin Kubernetes Configuration File

+ :package: admin ユーザーとしての認証に適した kubeconfig file を生成します

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
```

+ :package: WIP

```
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443
```
```
### 例

$ kubectl config set-cluster kubernetes-the-hard-way \
>   --certificate-authority=ca.pem \
>   --embed-certs=true \
>   --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443
Cluster "kubernetes-the-hard-way" set.
```

+ :package: WIP

```
kubectl config set-credentials admin \
  --client-certificate=admin.pem \
  --client-key=admin-key.pem
```
```
### 例

$ kubectl config set-credentials admin \
>   --client-certificate=admin.pem \
>   --client-key=admin-key.pem
User "admin" set.
```

+ :package: WIP

```
kubectl config set-context kubernetes-the-hard-way \
  --cluster=kubernetes-the-hard-way \
  --user=admin
```
```
### 例

$ kubectl config set-context kubernetes-the-hard-way \
>   --cluster=kubernetes-the-hard-way \
>   --user=admin
Context "kubernetes-the-hard-way" created.
```

+ :package: WIP

```
kubectl config use-context kubernetes-the-hard-way
```
```
### 例

$ kubectl config use-context kubernetes-the-hard-way
Switched to context "kubernetes-the-hard-way".
```



## 3. Verification

+ :package: WIP
  + ドキュメント差異があるので要確認

```
kubectl get componentstatuses
```
```
$ kubectl get componentstatuses
NAME                 AGE
scheduler            <unknown>
controller-manager   <unknown>
etcd-2               <unknown>
etcd-1               <unknown>
etcd-0               <unknown>
```

+ :package: WIP

```
kubectl get nodes
```
```
$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
worker-0   Ready    <none>   20h   v1.15.3
worker-1   Ready    <none>   20h   v1.15.3
worker-2   Ready    <none>   20h   v1.15.3
```


## 次のステップへ :rocket:

ここまでで、10. Configuring kubectl for Remote Access が完了です :raised_hands:

次は [11. Provisioning Pod Network Routes](./11-pod-network-routes.md) に進みます!! :muscle:
