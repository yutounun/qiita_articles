# 概要
ubuntuだとrpmが使えないため、うまくpg_hint_planが使えなかった。
そこでcentOS7を入れてrpm使ってインストールした。

記事ごとに割と言っていることがバラバラで本当に４日間ほどこのアンサーにたどり着くのに時間がかかった。就職以来一番時間をかけたエラーだと思う。

なんとか早く対象者にこの記事が見つかればと思う。

# 環境
```bash
(PostgreSQL) 13.5
centOS 7
```

# OS切り替え(CentOS7をvagrantで立ち上げる場合)
```bash
$ vagrant init
```
vagrantファイルを変更し、CentOS7を仮想環境に入れる設定

```Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
end
```

```bash
# 仮想環境立ち上げ。時間かかる
$ vagrant up
# 仮想環境接続できる
$ vagrant ssh
```
# rpmをvagrantに転送
[ここ](https://ja.osdn.net/projects/pghintplan/downloads/73862/pg_hint_plan13-1.3.7-1.el7.x86_64.rpm/)からインストールしたrpmをvagrant環境にコピーする

ゲストOSから下記を実行

```bash
# ホストOSからゲストOSにコピーするためにvagrant-scpプラグインをインストール
$ vagrant plugin install vagrant-scp
# コピー
$ vagrant scp pg_hint_plan13-1.3.7-1.el7.x86_64.rpm :/home/vagrant
```
# postgres13をインストール
仮想環境のvagrantユーザーで

```bash
$ sudo yum -y install epel-release yum-utils
$ sudo yum-config-manager --enable pgdg13
$ sudo yum -y install postgresql13 postgresql13-server
$ sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
$ sudo systemctl enable --now postgresql-13
# postgresが正常に動いているか確認
$ systemctl status postgresql-13
# postgresユーザーになる
$ sudo -i -u postgres
# DB操作できる
$ psql
```
## rpmインストーラーを使ってpg_hint_planをインストール
```$ exit```でvagrantユーザーになって、sudo権限で下記を実行

```bash
$ sudo rpm --install pg_hint_plan13-1.3.7-1.el7.x86_64.rpm
# 確認のためにもう一度
$ sudo rpm --install pg_hint_plan13-1.3.7-1.el7.x86_64.rpm
	package pg_hint_plan13-1.3.7-1.el7.x86_64 is already installed
```
## postgresqlに入って設定
```bash
$ psql
# db作成
postgres= create database test
# testデータベースを選択
postgres= \c test
postgres= LOAD 'pg_hint_plan';
LOAD
```
# 設定の更新
```bash
# だいぶ下の方にある。(Shift+gで一番下までひとっ飛び!!)
$ sudo vi /var/lib/pgsql/13/data/postgresql.conf
shared_preload_libraries = 'pg_hint_plan'
# 更新
$ sudo service postgresql-13 restart
```

# 確認
```bash
$ psql
postgres= \c test

postgres= 
/*+ SeqScan(users) */ 
EXPLAIN(ANALYZE,BUFFERS)
SELECT
count(*)
FROM
users;
```

# tips
ヒント句は、

- 統計情報を定期的に更新しない
- データ分布が変わらない
- 想定外の実行計画が選択された場合のコスト増大がすさまじい

の条件が揃った時は使うことを検討してもいいかもしれない
# 参考
[How To Install PostgreSQL 12 on CentOS 7 / CentOS 8](https://computingforgeeks.com/how-to-install-postgresql-12-on-centos-7/)

[pg_hint_planで実行計画を制御する](https://www.fujitsu.com/jp/products/software/resources/feature-stories/postgres/article-index/pg-hint-plan/)