# 11. Provisioning Pod Network Routes

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した Virtual Machine (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM) 

## この章でやること

+ Provisioning Pod Network Routes
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/11-pod-network-routes.md
+ 詳細
  + この章では、Node の Pod CIDR 範囲を Node の内部 IP アドレスにマップする、各 worker node のルートを作成します。

## 1. The Routing Table

+ :package: `kubernetes-the-hard-way` という名前の VPC network 内において、Routes を作成するために必要な情報を収集します

```
for instance in worker-0 worker-1 worker-2; do
  gcloud compute instances describe ${instance} \
    --format 'value[separator=" "](networkInterfaces[0].networkIP,metadata.items[0].value)'
done
```
```
$ for instance in worker-0 worker-1 worker-2; do
>   gcloud compute instances describe ${instance} \
>     --format 'value[separator=" "](networkInterfaces[0].networkIP,metadata.items[0].value)'
> done
10.240.0.20 10.200.0.0/24
10.240.0.21 10.200.1.0/24
10.240.0.22 10.200.2.0/24
```


## 2. Routes

+ :package: 各 worker instance 用の network routes を作成します

```
for i in 0 1 2; do
  gcloud compute routes create kubernetes-route-10-200-${i}-0-24 \
    --network kubernetes-the-hard-way \
    --next-hop-address 10.240.0.2${i} \
    --destination-range 10.200.${i}.0/24
done
```
```
### 例

$ for i in 0 1 2; do
>   gcloud compute routes create kubernetes-route-10-200-${i}-0-24 \
>     --network kubernetes-the-hard-way \
>     --next-hop-address 10.240.0.2${i} \
>     --destination-range 10.200.${i}.0/24
> done
Created [https://www.googleapis.com/compute/v1/projects/${Your Project Name}/global/routes/kubernetes-route-10-200-0-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20  1000
Created [https://www.googleapis.com/compute/v1/projects/${Your Project Name}/global/routes/kubernetes-route-10-200-1-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21  1000
Created [https://www.googleapis.com/compute/v1/projects/${Your Project Name}/global/routes/kubernetes-route-10-200-2-0-24].
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP     PRIORITY
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22  1000
```

+ :package: VPC network: `kubernetes-the-hard-way` のリストを確認してみましょう 

```
gcloud compute routes list --filter "network: kubernetes-the-hard-way"
```
```
### 例

$ gcloud compute routes list --filter "network: kubernetes-the-hard-way"
NAME                            NETWORK                  DEST_RANGE     NEXT_HOP                  PRIORITY
default-route-40b71b1cb409ae65  kubernetes-the-hard-way  0.0.0.0/0      default-internet-gateway  1000
default-route-7ad42a3d3d193df4  kubernetes-the-hard-way  10.240.0.0/24  kubernetes-the-hard-way   1000
kubernetes-route-10-200-0-0-24  kubernetes-the-hard-way  10.200.0.0/24  10.240.0.20               1000
kubernetes-route-10-200-1-0-24  kubernetes-the-hard-way  10.200.1.0/24  10.240.0.21               1000
kubernetes-route-10-200-2-0-24  kubernetes-the-hard-way  10.200.2.0/24  10.240.0.22               1000
```


## 次のステップへ :rocket:

ここまでで、11. Provisioning Pod Network Routes が完了です :raised_hands:

次は [12. Deploying the DNS Cluster Add-on](./12-dns-addon.md) に進みます!! :muscle:
