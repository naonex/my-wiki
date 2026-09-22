## 参考URL
<https://eng-blog.iij.ad.jp/archives/17746> ... WSL,Ubuntu導入手順で参考  
<https://docs.docker.com/engine/install/ubuntu/> ... Docker導入手順で参考

# wslでubuntu構築

## 1：Hyper-Vの有効化
Windowsのコントロールパネル->プログラムと設定->windowsの機能の有効かまたは無効に移動し、Hyper-Vにチェックする。  
コマンドプロンプトを管理者権限で実行し、 以下のコマンドを入力しPCを再起動する。
```
bcdedit /set hypervisorlaunchtype auto
```
※Hyper-V自体は使わないので、参考URLの手順でもOK想定

## 2：Ubuntu構築
マイクロソフトストアから以下アプリをインストール
* Windows Subsystem for Linux
* Ubuntu

インストール後、ターミナルを開いて最右部のプルダウンを押下し、Ubuntuを選択して起動する。  
起動すると初期設定が始まる。作成するユーザー名とパスワードを設定していく。（任意の設定でOK

初期設定が終わったらsystemdが利用できることを確認する。利用できなければ参考URLの手順で導入する。  
`systemctl --version`

## x：Ubuntuにプロキシ恒久設定を行う
home直下の .bashrc にプロキシexport設定を追加 ※WSLは、Windowsに設定されたプロキシ設定を継承できるようなので、printenvして設定済みなら不要...なはずだが、認証情報は継承されないので必要なケースあり
```
export http_proxy='http://{USER}:{PW}@{proxy_host}:{proxy_port}'
export HTTP_PROXY="${http_proxy}"
export https_proxy="${http_proxy}"
export HTTPS_PROXY="${http_proxy}"
```
sudoersに以下設定追加してsudo時にプロキシ設定を継承させる
```
# `sudo visudo` 後に以下の設定追加（エディタは環境によってviかnanoに変わる。使い方はググって調べて下さい）
Defaults env_keep += "http_proxy HTTP_PROXY https_proxy HTTPS_PROXY"
```
**※設定後、ターミナルを一度閉じて開き直すこと。プロキシ設定を完全に反映させるために必要となります**

# ubuntuにdocker構築

## 1：UbuntuにDockerインストール

以下手順を参考にdockerインストール  
<https://docs.docker.com/engine/install/ubuntu/>

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
（ここまでがdockerリポジトリ参照可能にするまでの手順

以下コマンドでdocker-engineインストール
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 2：Dockerにプロキシ設定
systemctlからエディタを開く
```
sudo systemctl edit docker
```

中央部に以下設定を追加する（エディタの使い方はnanoで調べてください）

---
```
[Service]
Environment="HTTP_PROXY=http://{USER}:{PW}@{proxy_host}:{proxy_port}"
Environment="HTTPS_PROXY=http://{USER}:{PW}@{proxy_host}:{proxy_port}"
```
**※以下コメントより上に追記する必要あり**
```
### Lines below this comment will be discarded
```

---
設定反映
```
sudo systemctl daemon-reload
```

Docker再起動
```
sudo systemctl restart docker
```

設定確認し、プロキシ設定が追加されてればOK
```
sudo docker info
```

ホームディレクトリで以下設定ファイルを作成する

`.docker/config.json` ※こっちはビルドしたコンテナに反映させる設定。この設定がないと `docker run` 内のプロキシ通信に失敗する
```
{
 "proxies":
 {
   "default":
   {
     "httpProxy": "http://proxy:8080",
     "httpsProxy": "http://proxy:8080"
   }
 }
}
```

## x：※念のため、dockerコマンドを一般ユーザーでも使えるようにしておく  
<https://docs.docker.com/engine/install/linux-postinstall/>

```
[dockerコマンドを一般ユーザーでも使えるようにする手順抜粋]
# 1. グループを作成（dockerインストールですでに作成されてる模様？）
# sudo groupadd docker
# 2. ユーザーをdockerグループに追加
sudo usermod -aG docker $USER
# 3. 再ログインして、dockerコマンドが使えるようになってることを確認
# 例：docker info ※プロキシ設定部とかが閲覧できればOK
```