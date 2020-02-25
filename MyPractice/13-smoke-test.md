# 13. Smoke Test

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Smoke Test
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/docs/13-smoke-test.md
+ 詳細
  + この章では、ここまでに作成してきた Kubernetes クラスターが正しく機能していることを確認することで、一連の作業を完了します。



## 1. Data Encryption

+ :package: `generic secret` を作成します

```
kubectl create secret generic kubernetes-the-hard-way \
  --from-literal="mykey=mydata"
```
```
### 例

$ kubectl create secret generic kubernetes-the-hard-way \
>   --from-literal="mykey=mydata"
secret/kubernetes-the-hard-way created
```

+ :package: etcd 内に入っている kubernetes-the-hard-way secret の hexdump を表示します

Print a hexdump of the kubernetes-the-hard-way secret stored in etcd:

```
gcloud compute ssh controller-0 \
  --command "sudo ETCDCTL_API=3 etcdctl get \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem\
  /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
```
```
### 例

$ gcloud compute ssh controller-0 \
>   --command "sudo ETCDCTL_API=3 etcdctl get \
>   --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/etcd/ca.pem \
>   --cert=/etc/etcd/kubernetes.pem \
>   --key=/etc/etcd/kubernetes-key.pem\
>   /registry/secrets/default/kubernetes-the-hard-way | hexdump -C"
00000000  2f 72 65 67 69 73 74 72  79 2f 73 65 63 72 65 74  |/registry/secret|
00000010  73 2f 64 65 66 61 75 6c  74 2f 6b 75 62 65 72 6e  |s/default/kubern|
00000020  65 74 65 73 2d 74 68 65  2d 68 61 72 64 2d 77 61  |etes-the-hard-wa|
00000030  79 0a 6b 38 73 3a 65 6e  63 3a 61 65 73 63 62 63  |y.k8s:enc:aescbc|
00000040  3a 76 31 3a 6b 65 79 31  3a ab ca 76 a0 ca 7b 28  |:v1:key1:..v..{(|
00000050  4a a9 0e 4c e0 0e c5 db  76 e8 05 20 de 4b 29 cd  |J..L....v.. .K).|
00000060  19 d7 6d 3b c7 9f 24 ae  a2 63 94 b5 5c 2a 0a 97  |..m;..$..c..\*..|
00000070  69 e3 f7 ac ad 9b 0f 07  15 de 19 e7 29 02 02 de  |i...........)...|
00000080  43 81 4f 50 58 86 e3 f6  ec 76 76 21 0d a0 f3 4f  |C.OPX....vv!...O|
00000090  87 dd 30 25 94 16 92 03  e4 5d 02 ed 5b df 06 b4  |..0%.....]..[...|
000000a0  a5 c5 03 70 62 ef 91 73  39 b5 ab 55 16 48 1d 46  |...pb..s9..U.H.F|
000000b0  08 f5 8b f5 5d 03 5d e0  6f 26 56 25 4c e7 1b 7c  |....].].o&V%L..||
000000c0  ee 52 19 14 65 e0 a5 a1  61 f3 37 3a a0 11 47 a9  |.R..e...a.7:..G.|
000000d0  fa 46 a2 bb a7 39 72 eb  40 d0 1c c4 3a 5c f9 8c  |.F...9r.@...:\..|
000000e0  89 32 32 c8 37 23 59 7c  c3 0a                    |.22.7#Y|..|
000000ea
```


## 2. Deployments

+ :package: nginx web server の deployment を作成します。

```
kubectl create deployment nginx --image=nginx
```

+ :package: nginx deployment によって作成された Pod のリストを確認します。

```
kubectl get pods -l app=nginx
```
```
### 例

$ kubectl get pods -l app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-554b9c67f9-hxc5n   1/1     Running   0          55s
```

### 2-1. Port Forwarding





























## 3. Services

+ WIP

```

```

## 4. WIP

+ WIP

```

```

## 5. WIP

+ WIP

```

```

## 次のステップへ :rocket:

ここまでで、13. Smoke Test が完了です :raised_hands:

次は [14. Cleaning Up](./14-cleanup.md) に進みます!! :muscle:
