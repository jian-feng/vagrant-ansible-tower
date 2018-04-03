# VagrantでAnsible Towerの仮想マシンを作りましょう

自分の環境に合わせて、`Vagrantfile`を変更し、`vagrant up`するだけで
Ansible Towerが使える状態になります。
所要時間、15 - 30分。

ソースは[github](https://github.com/jian-feng/vagrant-ansible-tower)上に公開しましたので、最初に`git clone`してください。

```bash
git clone https://github.com/jian-feng/vagrant-ansible-tower
cd vagrant-ansible-tower
```

## テスト済みバージョンの組み合わせ

- Mac OS: 10.13 (High Sierra)
- [virtualbox](https://www.virtualbox.org/wiki/Downloads): 5.1.34
- [Vagrant](https://www.vagrantup.com/downloads.html): 2.0.1
- [Ansible Tower](https://www.ansible.com/products/tower): 3.2.3
- [Ansible Engine](https://www.ansible.com/products/engine): 2.4.3(Ansible Coreです。Towerのインストーラに内包。)


## 自分の環境に合わせて調整

`Vagrantfile`を自分の環境に合わせて変更します。

```
vi Vagrantfile
```

Ansible Tower v3.2.3(2018/4/3時点最新版)をインストールするなら、
`$hostname`と`$private_ip`だけを変えれば良い。
異なるバージョンをインストールするなら、`download_url`と`install_file`も変更してください。

```ruby:Vagrantfile
$hostname = "tower-3-2-3"
$private_ip = "192.168.33.40"
$tower_bundle_download_url = "https://releases.ansible.com/ansible-tower/setup-bundle/ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"
$tower_bundle_install_file = "ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"
```

## 試用版ライセンスの準備(後でも良い)

[https://www.ansible.com/license](https://www.ansible.com/license)にアクセスし、試用版ライセンスファイルを申請しましょう。簡単な情報入力して、後ほど、ライセンスファイルがメールで送ってくれます。

> Workflow機能を使いたい場合は、"FREE ANSIBLE TOWER TRIAL - ENTERPRISE FEATURES"を選択してください。

> Towerエディションの違いは、[ここ](https://www.ansible.com/products/tower/editions)をご参照ください。

ライセンスファイルが届いたら、`lisence.txt`にリネームして、最初に`git clone`したディレクトリ(vagrant-ansible-tower)においてください。
自動インストールの際に、このライセンスファイルをTowerにセットしてくれます。

ライセンスファイルの届くのが遅れるかも知れませんので、先に進んでも大丈夫です。あとで、ブラウザでAnsible TowerのWeb GUIにアクセスしてライセンスをアップロードできます。

## いよいよ実行です


`vagrant up`を実行するだけです。

暫く(15 - 30分)すると、メッセージ"The setup process completed successfully"が出れば、Ansible Towerが使える状態となります。

```console
default: PLAY RECAP *********************************************************************
default: localhost                  : ok=131  changed=62   unreachable=0    failed=0
default: The setup process completed successfully.
default: Setup log saved to /var/log/tower/setup-xxxxxx.log
```


## 動作確認

- ブラウザで https://{前述の$private_ip} へアクセスしてください。
  - USER: admin
  - PASS: password

> ブラウザでアクセスの際に“この接続ではプライバシーが保護されません”という警告が表示されます。これは、Ansible Towerがデフォルトで自己証明書を使っているから。優しく許可してあげてください。

- プロジェクトを作成します。

プロジェクトタブにて、新規プロジェクトを以下のように作成します。

| 項目           |   設定値 |
|:--------------|:-----------|
|名前		| demo |
|SCMタイプ	| 手動  |
|PLAYBOOKディレクトリー| demo |
※記述以外の設定は初期値のままで構いません。

- ジョブテンプレートを作成します。

テンプレートタブにて、新規ジョブテンプレートを以下のように作成します。

| 項目           |   設定値 |
|:--------------|:-----------|
|名前		| hello
|インベントリ	| Demo Inventory
|プロジェクト	| demo
|PLAYBOOK	| hello.yml
|認証情報	| Demo Credential
※記述以外の設定は初期値のままで構いません。

- ジョブテンプレートを実行します。

テンプレートタブにて、先程作成したジョブテンプレートを実行して、
実行ステータスが”成功”であることを確認します。

## 続きは...

仮想マシン内のディレクトリ`/var/lib/awx/projects`がホスト側の`./projects`にSyncしていますので、ホスト側の好きなエディタで`./projects/`配下にサブディレクトリを作って、Playbookを開発すれば、Ansible Towerでも見えるになります。

以下のコンテンツを参考しながら、練習しましょう。
- [日本語概要:Ansible Tower利用ガイド](https://github.com/h-kojima/ansible/tree/master/ansible-tower)
- [英語ワークショップ:ansible/lightbulb](https://github.com/ansible/lightbulb/tree/master/guides/ansible_tower)

## 仮想マシンが不要になったら

- `vagrant halt`を実行して、仮想マシンをシャットダウンします。再度`vagrant up`で起動できます。
- `vagrant destroy`を実行して、仮想マシンを削除します。

## 補足: Enable remote access to postgresql

You can ssh into guest vm as user vagrant (`vagrant ssh`), then run command below.

```sh
# Enable remote access
echo "host  all  all  0.0.0.0/0  md5" | sudo tee -a /var/lib/pgsql/9.6/data/pg_hba.conf
# Restart service
sudo systemctl restart postgresql-9.6
```

Now, you can access awx db as from host os.

```sh
psql -h {前述の$private_ip} -W awx awx
# Password is password
```
