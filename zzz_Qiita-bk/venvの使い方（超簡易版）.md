### 環境構築
- 仮想環境を作成します

`python -m venv venv1`

作成後、以下の様なコマンドで仮想環境に入る

`.\venv1\Scripts\activate`

- pipコマンドで必要なパッケージをインストールします

`pip install lxml`

#### おまけ
仮想環境から出る時は`deactivate`  
`pip freeze`でインストールパッケージ確認（仮想環境にちゃんと入れてるかの確認）  
`pip uninstall lxml`のようなコマンドでアンインストール
