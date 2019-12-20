# 00. Prepare

## GCP プロジェクト

+ GCPプロジェクト: `iganari-k8s-hardway-pre` を使用する想定で書いていきます。
  + 該当の箇所は適宜読み替えて下さい。

## 実行環境

+ 実行環境を隔離するために、 VM を作成します。
+ 作成する GCE
  + プロジェクト: iganari-k8s-hardway-pre
  + ネットワーク: デフォルトの VPC ネットワーク
  + name: kubernetes-the-hard-way-manual

+ GCP と認証を通します。

```
gcloud auth login
```

+ gcloud コマンドの設定を行います。

```
gcloud config set project iganari-k8s-hardway-pre
gcloud config set compute/zone asia-northeast1-c
```

+ GCE を起動します。

```
gcloud beta compute instances create kubernetes-the-hard-way-manual \
    --machine-type=f1-micro \
    --subnet=default \
    --image=ubuntu-1804-bionic-v20191211 \
    --image-project=ubuntu-os-cloud 
```

+ この GCE に SSH して作業を行います。

```
gcloud compute ssh kubernetes-the-hard-way-manual
```

+ Debian と Ubuntu 用のクイックスタート
  + https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu?hl=ja
  + 上記を参考に、 Ubuntu に gcloud を作成する

+ Ubuntu の整備の準備をします。

```
apt update
apt upgrade -y
apt dist-upgrade -y
apt autoremove -y
```

+ User を追加します。
  + デフォルトだと root ユーザしかいないためです。

```
export _user_name='iganari' 

useradd -m -s /bin/bash ${_user_name}

echo "${_user_name} ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/${_user_name}
chmod 0440 /etc/sudoers.d/${_user_name}
```

+ User 変更をします。

```
su - ${_user_name}
```


+ gcloud コマンドのインストール
  + https://cloud.google.com/sdk/docs/quickstart-debian-ubuntu?hl=ja

```
export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update && sudo apt-get install -y google-cloud-sdk
```

+ gcloud コマンドの設定を作る

```
export _pj='kubernetes-the-hard-way-manual'

gcloud config configurations create ${_pj}
gcloud config set project ${_pj}
gcloud config configurations list
```
```
### 例

$ gcloud config configurations list
NAME                            IS_ACTIVE  ACCOUNT  PROJECT                         DEFAULT_ZONE  DEFAULT_REGION
kubernetes-the-hard-way-manual  True                kubernetes-the-hard-way-manual
```


ここまでで、 00. Prepare が完了です :raised_hands:

+ 作業が終わったら、 GCE は停止しておきましょう

```
gcloud beta compute instances stop kubernetes-the-hard-way-manual
```
