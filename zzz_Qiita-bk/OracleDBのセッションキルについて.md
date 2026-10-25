TNSにTCP/IPのkeepAlive設定を組み込んだ後に、訳あってSQLdeveloperの強制終了とかをした時、なかなかセッションキルができなかったので備忘録として残しておきます。

参考リンク  
①<https://communities.curl.com/showthread.php?tid=1084>  
②<https://dekiruengineer.com/engineer/how_to_find_os_processes_oracle_killing_session/>

# keepAlive設定について
TNSに ENABLE=BROKEN を追加すると、OSのTCP/IPのKeepAliveが有効となる

```
XXXDB =
  (DESCRIPTION =
    (ENABLE=BROKEN) ←追加
    (ADDRESS_LIST =
～～～
  )
```

#### TCP/IPのkeepAlive設定
##### Windowsの場合※レジストリ設定の反映には要再起動なので注意
```
レジストリ HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
KeepAliveTime(DWORD 32ビット) の値
デフォルト: 7200000 (ミリ秒)
```

##### Linuxの場合
```
/etc/sysctl.conf の net.ipv4.tcp_keepalive_time の値
デフォルト: 7200 (秒)
```

# 肝心のOracleDBセッションキルについて
最初はSQLdeveloperのセッションモニターから、対象セッションを選択して右クリック→セッションの中断でキルしようとした（keepAlive設定前はだいたいこれだけでOKだった）が、一度実行した後もゾンビみたいなセッションが残り続けてしまう現象が発生。←多分SQL実行中に強制終了したせい…  
再度セッションの中断を実行するが、ユーザーIDがない？とか言われてエラーとなり、セッションキルができない事態に。。

ググったところ、文頭記載の参考リンクにたどり着いた。  
windowsマシンのoracleDBなら、①リンク先のorakillコマンドで停止できるそうだが、生憎linuxマシンのoracleDBだったので、②リンク先のプロセスキル方式で対応した。  
その際のプロセスID確認SQLは①リンク先のものを使用して上手くいきました。引用先様ありがとうございます！ :bow: 
