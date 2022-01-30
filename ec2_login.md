# 目的
ec2のインスタンスにログインする時に、特に久しぶりだとコマンドを忘れたり、覚えていてもパブリックDNS探すのが面倒だったり、そもそもこの長いコマンドを打つのが嫌だった。。。

```
$ ssh -i <key_name> ec2-user@<Public_DNS>
```

まさかこれを簡単にする方法があるとは知らなかった。
# 前提
- EC2インスタンスを起動済み
- ElasticIPをそのインスタンスにアタッチ済み

# 概要

# .sshディレクトリ を作成し、権限を付与する
```bash
$ mkdir ~/.ssh
# 所有者に全ての権限を与える
$ chmod 700 ~/.ssh
```

# key.pemを.sshディレクトリに移動し、権限を付与する
EC2インスタンス起動時にダウンロードした自分のキーをkey.pemに置き換えること。
所有者にread権限のみ与える。

```bash
$ mv key.pem ~/.ssh
$ chmod 400 ~/.ssh/key.pem
```

# configにSSH接続情報を登録しておく
configファイルがsshディレクトリ以下になければ作成する
```
$ touch ~/.ssh/config
$ vi ~/.ssh/config
```

```~/.ssh/config
host ec2_login
  HostName <PublicDNS>
  Port 22
  User ec2-user
  IdentityFile ~/.ssh/key.pem
```

HostNameにはインスタンス詳細に記載の自分ものを。
Userはいつもsshする時に使うUser名。
IdentityFileは前述で移動したkeyを。

# configファイルの権限
所有者のみread/write権限を与える
```
$ chmod 600 ~/.ssh/config
```

# ssh接続してみる
```bash
$ ssh ec2_ec2_user
Last login: Sun Jan 30 21:56:11 2022 from softbank219059166010.bbtec.net

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
```

# できた！！