# 06. Generating the Data Encryption Config and Key

## 注意点

+ アイコンの説明

アイコン | 説明
:-: | :-
:computer: | ホストマシン
:package: | Google Compute Engine (GCE) 上に作成した踏み台サーバ (VM)
:police_car: | GCE 上に作成した Kubernetes Control Plane 用のサーバ (VM)
:car: | GCE 上に作成した Kubernetes Nodes 用のサーバ (VM)

## この章でやること

+ Generating the Data Encryption Config and Key
  + https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/06-data-encryption-keys.md
+ 詳細
  + WIP

## 1. The Encryption Key

+ :package: 暗号化キーを作成します

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```
```
### 例

$ echo $ENCRYPTION_KEY
SFVeSFv7DrHbTRKg7I6iZ+w2nuJUhjUPgVqkGlFA3IM=
```

## 2. The Encryption Config File

+ :package: 暗号化キーの設定ファイルである `encryption-config.yaml` を作成します。

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

+ :package: 作成した `encryption-config.yaml` を、 各 controller instance (:police_car:) にコピーします。

```
for instance in controller-0 controller-1 controller-2; do
  gcloud compute scp encryption-config.yaml ${instance}:~/
done
```
```
### 例

$ for instance in controller-0 controller-1 controller-2; do
>   gcloud compute scp encryption-config.yaml ${instance}:~/
> done
encryption-config.yaml                                                                          100%  240   415.6KB/s   00:00
encryption-config.yaml                                                                          100%  240   334.8KB/s   00:00
encryption-config.yaml                                                                          100%  240   280.1KB/s   00:00
```

## 次のステップへ :rocket:

ここまでで、 06. Generating the Data Encryption Config and Key が完了です :raised_hands:

次は [07. Bootstrapping the etcd Cluster](./07-bootstrapping-etcd.md) に進みます!! :muscle:
