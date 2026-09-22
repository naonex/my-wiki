* 前提条件
    * Docker環境構築済み

## GitLab構築編
docker-compose.ymlの階層で以下コマンド実行  
（同階層にファイル配置してるので、Docker利用マシンの任意階層にコピーして利用してください [[docker-compose.yml]]）

```
# ビルド＆コンテナ作成（初回のみ実行。一度コンテナ作成したら以降は起動から実行でOK）
docker compose up -d
# ビルド＆コンテナ作成後、一定時間経つとコンテナが落ちるケースあり（詳細は後述のNOTE参照）
# この場合、 `docker ps` でコンテナ名でてこなかったらコンテナ起動コマンドを実行して再起動する

# コンテナ起動
docker compose start

# コンテナ停止（マシンシャットダウンする前に手動実行することを推奨 ※次回起動時に後述の復旧コマンドが必要になる可能性あり）
docker compose stop
```

実行後、ブラウザよりlocalhostのhttpsにアクセス <https://localhost>

NOTE: 初回起動時は以下コマンド実行しないと正しくgitlabが起動できないケースあり。  
また、 `docker compose stop` せずにマシンシャットダウンした時なども以下コマンド実行しないと復旧できないケースがあります。

`docker exec -it gitlab-web-1 rm /opt/gitlab/var/unicorn/unicorn.pid && docker restart gitlab-web-1`

## GitLabインポート編
デフォルト管理者ユーザー：root、パスワードは初回アクセス時に設定

gitlabのプロジェクトエクスポートでtar.gzファイルを出力しておく

移行先のローカルgitlabでプロジェクト作成 > プロジェクトインポートでgitlabエクスポートを選択してインポートする