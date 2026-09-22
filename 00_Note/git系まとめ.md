##### 強制push（※この際、ログインしているgitユーザーのパスワードを聞かれる）

```bash
git push -f origin ブランチ名

# 通常はこっちでOK
git push -f
```

##### rebase

```bash
git rebase -i HEAD~1

# こっちのほうが自分ローカル分しかでないからいい
git rebase -i
```

##### 現在のコミットのメッセージを編集（※タイムスタンプが更新されて差分があることになる時もあるので注意！！）

```bash
git commit --amend
```

##### githubと連携するためのプロキシ設定（http.proxyだけでよさそう。localリポジトリにのみ適用する設定だと上書きされない）

```bash
git config --local http.proxy http://xxx.xxx.xxx.xxx:xxxx
#git config --global https.proxy http://xxx.xxx.xxx.xxx:xxxx
#git config --global url."https://".insteadOf git://

# 以下で解除できるらしい
git config --global --unset http.proxy
#git config --global --unset https.proxy
#git config --global --unset url."https://".insteadOf git://
```

##### gitDiffでsjis差分を見るための設定

```bash
# global attributes設定
git config --global core.attributesFile C:\\Users\\user1\\work\\git\\.gitattributes
```

.gitattributesにこんな設定する

```
*.html diff=sjis
```

global diffフィルタ設定

```bash
# cp932はwindows用sjis　win環境ならこっちのほうがよい
#git config --global diff.sjis.textconv "iconv -f sjis -t utf-8"
git config --global diff.cp932.textconv "iconv -f cp932 -t utf-8"
```
