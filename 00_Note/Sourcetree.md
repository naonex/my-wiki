# WSL内のディレクトリをSourcetreeから参照する場合

#### 参考URL
- <https://note.com/ready_snail223/n/n4a110f078564>
- <https://qiita.com/num-bx77/items/76a679101026b5681895>

## 設定備忘録

```
# 個別設定（この設定が理想だが、警告が出る）
git config --global --add safe.directory "%{prefix}///wsl$/Ubuntu/home/${USERNAME}/wslDevelopment/laravel-app"

# 個人PCなので、セキュリティ無効化で設定しておく
git config --global --add safe.directory '*'
```
