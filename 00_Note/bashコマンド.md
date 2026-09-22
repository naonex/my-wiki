■ファイルの状態を再帰的に確認

```bash
ll -d $(find ./)
```

■ディレクトリ同士を比較し、差分のあるなしだけ表示

```bash
 diff -rqs testScript/chk_Life/dummy1
```

■なにかキーが押されるまで待機

```bash
read -p "Hit enter: "
```

無限ループ

```bash
# 無限ループ
while :
do
    echo "ループ中"
done

# ワンライナーだとこんな感じ
while true; do date;sleep 1; done
```

コマンド結果をdiffる

```bash
diff <(ll /home/user/backup/testDir/) <(ll /etc/appDir/)
```

引用符を含むコマンドをスクリプトで実行するとき

```bash
  CMD=(rsync -av -e 'ssh -o StrictHostKeyChecking=no' $SSHUSER@$IPADDR:$LOGDIR/ $OUTDIR)
  "${CMD[@]}"
```
