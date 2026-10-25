# はじめに
クローン元と平行稼働するつもりなら、「すべてのネットワークアダプターでMACアドレスを生成」してクローンするのが無難と思われる

# MACアドレスが振り直される故に発生する問題を解決する

## クローンするとeth*がカウントアップされていく問題
参考URL：<https://blog.lowaivill.com/development/eth0_device/>

`/etc/udev/rules.d/70-persistent-net.rules`にクローン元の記載があるせい  
NICの分だけ新規追加されているので、削除してNAMEのeth番号を振り直そう

※設定後、要再起動

```
reboot
```

## ifcfg-eth*ファイルのMACアドレスを変更
対象ファイル：`/etc/sysconfig/network-scripts/ifcfg-eth*`※NICの数だけあるはず   
振り直されたMACアドレスはVirtualBoxのGUIから確認できる  
固定IPを使うのであれば、このタイミングでIPも変えておこう  
UUIDは振ってない場合自動設定してくれるので、設定されてる場合はコメントアウトとかしておくでOK

※設定後、要ネットワーク再起動

```
service network restart
```

ここまでの設定で、sshクライアントから接続できるようになるでしょう

# サーバー名を変更する
`/etc/sysconfig/network`のHOSTNAMEを変更

※設定後、要再起動

# `/etc/hosts`を設定

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
<IPアドレス> <ホスト名>
```

とりあえず現時点で思い浮かぶものはこれくらいかな？
