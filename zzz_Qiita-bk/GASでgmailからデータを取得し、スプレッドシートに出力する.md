# 初めに
gmailに飛んでくる大量のアラートメールを分析したいという動機からスタート  
いろいろ調べていくうちにGASを使用することで、

* gmailからデータを取得
* googleスプレッドシートにデータを出力
* （GASでデータを加工）

ができそうだと分かった。

## gmailからデータを取得する
取得自体は結構簡単にできた。だが、検索条件に一致しないデータも取得してる？また、すべてのメールが取得できてなくない？と感じて、いろいろ調べてみたところ、GASのgmail検索は、「検索条件にヒットしたメールが存在するスレッド単位（件名でまとめられた単位）でメールデータを取得している＆一度に取得できるスレッドは500まで」ということが分かった。なので、否応なしに余分なデータもついてきてしまうという仕様・・・（出力先のスプレッドシートでフィルタを掛ければいいんだけどね・・・）  
それと検索条件に一致するメールデータを全量取得するロジックは、結構悩まされた。  
是非参考にして欲しい。

```javascript
searchQuery = "検索条件"
loopCount = "ループ上限（※無限ループにしたら間違いなく全量取得できるけど、GASからgmailの取得には上限があるので注意）"

  /* 自身のGmailからクエリ条件と一致するメール（スレッド）を全て取得する */
  var allThreads = [];
  var max = 500;
  //検索上限スレッド数に達するまでループ
  for(var i = 0; i < loopCount; i++){
    //0,500で0~499のスレッドが取得されるので、次に取得するのは500から
    var threads = GmailApp.search(searchQuery, i*max, max);
    if (threads.length <= 0) break;
    //取得したスレッドをオールに結合
    Array.prototype.push.apply(allThreads, threads);
  }
```

### googleスプレッドシートにデータを出力する準備
上で取得したスレッドからメール情報を取得して、スプレッドシート出力用配列にデータを格納する

```javascript
  //データ格納用変数
  var valMsgs = [];
  //見出し行セット
  var frozenRow = 2;
  valMsgs.push([searchQuery,'','','']);
  valMsgs.push(['Date','From','Subject','PlainBody']);
  
  for(var n in allThreads){
    var thread = allThreads[n];
    var msgs = thread.getMessages();
    
    for(m in msgs){
      var msg = msgs[m];
      
      var date = msg.getDate();
      var from = msg.getFrom();
      var subj = msg.getSubject();
      var body = msg.getPlainBody();

      valMsgs.push([date,from,subj,body]);
    }
``` 

## googleスプレッドシートにデータを出力する
これもデータを出力するだけなら結構簡単だった。だが、GASでフォーマットを整えるとなるとちょっとややこしい。  
今回私が使用したのは、

* 見出し行の作成
* 区切り文字で分割
* 日付データフォーマットをセット
* 空白列を削除（自作ロジック）

などである。excelでできるようなことはたいていできそうなので、挑戦し甲斐があるなと思った。  
空白列の削除はAPIとして用意されてても良かったんじゃないかなーと感じた。  
まあそこまで難しくなかったけどね。。

```javascript
  /* スプレッドシートに出力 */
  if(allThreads.length > 0){
    var sheetName = SpreadsheetApp.getActiveSpreadsheet().getName();
    var dateFormat = "yyyy/MM/dd HH:mm:ss"
    var nowTime = Utilities.formatDate(new Date(), "JST", dateFormat);
    
    //mainスプレッドシートに新規シートを作成
    var activeSheet = SpreadsheetApp.getActive().insertSheet("出力結果 (" + nowTime + ")");
    
    activeSheet.setFrozenRows(frozenRow);  //見出し行設定
    activeSheet.getRange((1 + frozenRow), 1, (valMsgs.length - frozenRow)).setNumberFormat(dateFormat);  //日付列にフォーマット設定
    activeSheet.getRange(1, 1, valMsgs.length, 4).setValues(valMsgs);  //メールデータセット
    //PlainBody分割処理（改行で分割）
    activeSheet.getRange((1 + frozenRow), 4, valMsgs.length).splitTextToColumns(String.fromCharCode(10));
    //空白列削除
    for (var i = activeSheet.getLastColumn(); i > 0; i--) {
      if (activeSheet.getRange(1, i, valMsgs.length).isBlank()) {
        activeSheet.deleteColumn(i);
      }
    }
```

## GUIから簡単に使用できる形にしたい
excelマクロのようにGUIから簡単に使用できる形にしたかったので、スプレッドシートにボタン替わりとなる適当な画像を挿入し、画像にスクリプトを割り当てる。これで画像をクリックしたらスクリプトを実行できるようになる。  
また、スプレッドシートに名前の定義を設定しておくことで、GASからその定義を使用してセル情報を取得できるようになる。

（画像をクリックしたら以下のファンクションが実行されるようにしました）

```javascript
function clickOutputStartButton() {
  var ui = SpreadsheetApp.getUi();
  var mainSheet = SpreadsheetApp.getActiveSheet();
  //スプレッドシートの名前付き定義で取得
  var searchQuery = mainSheet.getRange('searchQuery').getValue();
  var loopCount = mainSheet.getRange('loopCount').getValue();
  
  var result = ui.alert(
    'gmailから以下の検索条件で出力します',
    searchQuery,
    ui.ButtonSet.OK_CANCEL);

  if (result == ui.Button.OK) {
    var startTime = new Date();
    //メール出力処理実行
    var ret = outputMail(searchQuery, loopCount);
    var endTime = new Date();
    mainSheet.getRange('outputTime').setValue((endTime - startTime) / 1000);
    
    //値が返ってきたかで判定
    if (ret != null) {
      var result = ui.alert(
        '完了',
        'gmailからの出力に成功しました',
        ui.ButtonSet.OK);
      
    } else {
      var result = ui.alert(
        '警告',
        '検索条件に一致するメールはありませんでした',
        ui.ButtonSet.OK);
    }
  }
}
```

## とりあえず完成したけど・・・遅い！！
以上で要件を満たすツールは完成したかな？と思ったが、メールデータの出力までに結構時間が掛かる・・・  
データ量が多ければ仕方ないが、時は金なりというし、極力スピードアップさせたい。

調べてみたら時間が掛かっていた処理は、gmailから取得したデータをスプレッドシート出力用の配列としてまとめる箇所だった。  
１スレッドずつメールを取得するロジックがまずいのかと思ったので、スレッド配列を渡したらまとめてメッセージを返してくれるファンクションを発見し、早速使用してみる・・・  
が、こちらを使用したほうが遅くなりました＾＾；

javascriptだとPromiseとかで並列処理ができるようだが、現時点でのGASではPromiseを使用できない模様。  
スプレッドシート⇔GAS⇔gmailの構成だと自分ではこの辺が限界のようだ・・・

しかし、調べていくうちにGASでwebアプリを作れるということが分かったので、GAS内のhtmlにjavascriptを埋め込めば並列処理できそう。  
こちらは今度試してみよう・・・

### GASの速度改善を実施してみて分かったこと
GASを速くするためには・・・

* 基本はAPIのコール回数を減らす
* ただし、一度に大量のデータを処理する場合、逆に時間が掛かるようになってしまう時もある

まとめると、**サーバー側に負荷を掛けないほうが早い**と言える

# 最後に

以下に私が作成したツールを閲覧専用として公開しておきます。  
googleアカウントでログインして「ファイル」→「コピーを作成」から自身のgoogleドライブに取り込んで使用できます。  
怪しいコードが仕込まれてないか心配な人はスクリプトエディタでソースを確認してから使ってね☆彡

**gmail検索ツール_公開用**
<https://docs.google.com/spreadsheets/d/1Z1LJ1IJCytDbTuazhy2tyVtAb0JJ-mPM9lPaUmLVEO4/edit?usp=sharing>

**gmailSearchScript（上記ツールが使用するスクリプトのソース）**
<https://github.com/naonex/gmailSearchScript>
