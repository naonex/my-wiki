#### 容量を調べる

```bash
# サーバ全体
df -Th
df -h

# ディレクトリサイズ
du -sh /home/user1/*

# ファイルサイズ
ls -lh
```

##### メモリ使用率確認

```bash
sar -r 3 500
```

##### CPU使用率確認（us ユーザーが使用したCPUの割合（%））

```bash
vmstat 5
sar
```

#### rsync

```bash
# ディレクトリのタイムスタンプを無視
# man rsync より
# -O, --omit-dir-times        omit directories from --times

# 所有者IDとグループID以外を引き継ぎ(rlptD)＆実行内容出力(v)＆テストラン(n)
rsync -rlptDvn remoteUser1@xxx.xxx.xxx.xxx:/home/remoteUser1 ./

# ファイル転送テンプレ（全情報引き継ぎ(a)＆実行内容出力(v)＆テストラン(n)）※送信先ディレクトリは省略できない模様？
rsync -avn /home/user1 remoteUser1@xxx.xxx.xxx.xxx:/home/remoteUser1

# プル
rsync remoteUser1@xxx.xxx.xxx.xxx:/var/log/target.log .
```

##### jenkinsでansibleのログを色付きで見るために必要な環境変数

```bash
export ANSIBLE_FORCE_COLOR=true
```

##### ユーザーのホームディレクトリ変更

```bash
usermod -d /home/new-home sample
```

##### ホストネーム変更（centos6）

```bash
vi /etc/sysconfig/network
```

##### ネットワーク手動起動コマンド

```bash
ifup eth3
```

##### パスワード等を打ち間違えた時
`「［Ctrl］＋［U］キー」で、入力をリセットできる（それまでに入力した文字を全削除）`

##### 送信元を指定してping

```bash
ping xxx.xxx.xxx.xxx -I yyy.yyy.yyy.yyy
```

##### 使用しているポートを調べる

```bash
lsof -i
lsof -i:8080
```

##### 複数ファイルの中身検索

```bash
find ./ -type f -print | xargs grep 'targetString'
```

##### ファイル検索例

```bash
find / -name "*jenkins*"

# 検索したファイルに対してコマンド実行する時
find ./ -name "*.sh" | xargs chmod +x
```

##### 時刻同期設定

```bash
cat /etc/ntp.conf
```

##### linuxバージョン確認

```bash
cat /etc/redhat-release
# 以下はOSのbit数確認用
getconf LONG_BIT
```

##### 正規表現で削除

```bash
rm -rf *MIZUHO_ORDER*
```

##### 環境変数を読み直してスイッチ

```bash
su -l
```

##### 一定時間毎にコマンド実行（ログ出力付き）

```
watch -d 'bash /home/testUser/tools/check.sh | tee -a /home/testUser/tools/check.log'
```
