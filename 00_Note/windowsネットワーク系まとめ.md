##### リモートデスクトップでコピペできなくなった時は、リモート先で以下のアプリを再起動する
`rdpclip.exe`

##### インターネットオプション検索名（検索して実行すれば楽に起動できる）
`inetcpl.cpl`

---

#### ルーティング設定

確認

```
route print
```

設定コマンド例　（接続先　ネットマスク　ゲートウェイ）

```
route -p add xxx.xxx.xxx.xxx mask 255.255.255.255 zzz.zzz.zzz.zzz
route -p add xxx.xxx.xxx.xxy mask 255.255.255.255 zzz.zzz.zzz.zzz
```

※固定ルート削除の際は「route delete *」を使用しないこと
デフォゲ宛の固定ルートも削除されてしまう

---

#### 送信元を指定してping

```
ping -S xxx.xxx.xxx.xxx yyy.yyy.yyy.yyy
```

---

#### NTPクライアント設定
`gpedit.msc`  
参考URL：<https://www.server-world.info/query?os=Windows_Server_2012&p=ntp&f=2>

#### NTPサーバー設定
<https://www.eaton-daitron.jp/techblog/5751.html>
