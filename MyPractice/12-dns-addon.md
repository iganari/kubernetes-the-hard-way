# 12. Deploying the DNS Cluster Add-on

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Deploying the DNS Cluster Add-on
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/12-dns-addon.md
+ 詳細
  + WIP

## 1. The DNS Cluster Add-on

+ :package: K8s クラスターの拡張機能 (add-on) である `coredns` をデプロイします

```
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
```
```
### 例

$ kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

+ :package: `kube-dns` deployment によって、生成された Pod のリストを確認します

```
kubectl get pods -l k8s-app=kube-dns -n kube-system
```
```
$ kubectl get pods -l k8s-app=kube-dns -n kube-system
NAME                     READY   STATUS    RESTARTS   AGE
coredns-5fb99965-92xqh   0/1     Running   0          34s
coredns-5fb99965-ktxpr   0/1     Running   0          34s
```

## 2. Verification

+ :package: `busybox` の deployment を作成します

```
kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
```
```
$ kubectl run --generator=run-pod/v1 busybox --image=busybox:1.28 --command -- sleep 3600
pod/busybox created
```

+ :package: `busybox` deployment によって、生成された Pod のリストを確認します

```
kubectl get pods -l run=busybox
```
```
### 例

$ kubectl get pods -l run=busybox
NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   0          43s
```

+ :package: `busybox` Pod のフルネームを取得します

```
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```
```
### 例

$ echo ${POD_NAME}
busybox
```

+ `busybox` Pod の中から kubernetes service の DNS lookup を実行します。

```
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```
```
### 例

$ kubectl exec -ti $POD_NAME -- nslookup kubernetes
Error from server: error dialing backend: x509: certificate is not valid for any names, but wanted to match worker-1

---> 参照出来ていない…
```

## 次のステップへ :rocket:

ここまでで、12. Deploying the DNS Cluster Add-on が完了です :raised_hands:

次は [13. Smoke Test](./13-smoke-test.md) に進みます!! :muscle:
