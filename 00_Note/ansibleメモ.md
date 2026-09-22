#### 指定したホスト（グループ）に実行させる（コマンドオプションで対象ホストを上書きする）

```bash
ansible-playbook /etc/ansible/service/tomcatStatus.yml --limit api001
ansible-playbook /etc/ansible/service/tomcatStatus.yml --limit api002

sudo -u ansible-user -i env ansible-playbook ./release/releaseCheck.yml -D --limit api004
```

---

#### リストの要素数を調べる
```yaml
- name: test
  debug: msg="{{ result.stdout_lines | length }}"
```

#### playbookカレントディレクトリ
```yaml
# {{playbook_dir}}=実行中のplaybookがある階層
template_path: "{{playbook_dir}}/template"
```

#### 1台ずつ実行していく設定
```yaml
# （sshパスワード接続時にrsyncのパスワード入力応答エラーを回避する設定（対象ホスト複数時）
# 　※sshのパスワードが違う時にそれぞれ入力できるわけではない）
  serial: 1
```

ansible.cfgファイル

```
# インベントリファイル
#hostfile = ./hosts

# タスクの引数を表示させる設定
#display_args_to_stdout = True

# 暗号化ファイルを複合化するためのパスワード入力プロンプトを常に表示させる設定
#ask_vault_pass = True
```

ユーザー作成playbook（パスワード設定）

```yaml
# ※パスワード設定を行っているため、常にchangedで表示される
- name: mainteユーザー作成
  user: 
    name: mainte
    password: "{{ var_mainte_password | password_hash('sha512') }}"
```

#### ansiblie変数名について
※ansibleは変数名にハイフン(-)を使えない

##### sudoers

```bash
# jenkinsから実行させるなら権限設定しておく
Defaults:jenkins !requiretty
jenkins ALL=(ansible-user) NOPASSWD: /bin/bash
```
