Linux環境でファイルから特定文言を抽出していくコマンドサンプル（巨大ファイルゆえエディタから開けないときとか）

```
sed -n       1,1000000p /usr/log/bigLog.log | grep TARGETSTR >> /home/test1.txt
sed -n 1000001,2000000p /usr/log/bigLog.log | grep TARGETSTR >> /home/test1.txt
sed -n 2000001,3000000p /usr/log/bigLog.log | grep TARGETSTR >> /home/test1.txt
```
