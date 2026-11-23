# 環境構築手順（venv版）

## 前提条件
- python 3.14以上
- node.js 22以上

#### プロジェクトカレントディレクトリでpython仮想環境作成
`py -m venv .venv`

#### 仮想環境に入る
`.venv\Scripts\activate`

#### django含む必要なライブラリインストール
(venv) `pip install -r requirements.txt`

#### DBマイグレーション
(venv) `python djangoApp\manage.py migrate`

#### フロント資材ビルド
(venv) `npm --prefix frontend run build`

#### 付録：管理者アカウント作成
(venv) `python djangoApp\manage.py createsuperuser`

---
## djangoサービス起動 ※以下はデバッグモード起動になる
(venv) `python djangoApp\manage.py runserver` ...（ctrl + c で終了）
#### ポート指定したい時の例：
(venv) `python djangoApp\manage.py runserver 8080` ...（ちなみにデフォだと8000で起動）

## ※デバッグモード起動時は、以下でフロント資材を提供する必要あり
(venv) `npm --prefix frontend run dev`

## NOTE: vscodeの実行とデバッグ構成に便利な開始設定定義してるのでそちらを利用するのを推奨

`localhost:8000(デフォの場合)` にアクセスし、ログインページに行けたらOK。  
ログインして確認してね！アカウントはローカルDB管理なので適当に作成してもらってOKです！

---
# ※初回構築時のコマンド例

#### djangoインストール
(venv) `pip install Django`

#### 使用するライブラリインストール
(venv) `pip install django-vite`

##### 使用ライブラリを確認し、requirements.txtを作成
(venv) `pip freeze | Out-File -FilePath requirements.txt -Encoding utf8`

#### プロジェクトを作成する
(venv) `django-admin startproject djangoApp`

#### アプリケーションをつくる
(venv) `python djangoApp\manage.py startapp ffmpeg`
