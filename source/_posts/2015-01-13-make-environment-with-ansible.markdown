---
layout: post
title: "環境構築Ansibleでやってみた"
date: 2015-01-13 23:49:25 +0900
comments: true
categories: ["ansible", "playbook", "chef", "vagrant", "virtual environment"]
---
Chefは挫折してしまったので、Ansibleで再度環境構築を・・・。

##Vagrantを使ってサーバー準備

###Vagrantfileを作成
```
$ vagrant init chef/centos-6.5
```

###Vagrantfile編集

```
 # config.vm.box = "chef/centos-6.5" #コメントアウト

  config.vm.define "web" do |node|  
    node.vm.box = "chef/centos-6.5"  
    node.vm.hostname = "web"  
    node.vm.network :private_network, ip: "192.168.59.34"  
  end
````
  
###仮想環境立ち上げ
```
Vagrant up
```
回線が微妙なので長い・・・。

##鍵の作成

すでにあるんで、これでOK？
```
cat .ssh/id_rsa.pub | ssh vagrant@192.168.59.34 "cat >> .ssh/authorized_keys”

echo "[web]" > hosts

echo "192.168.59.34" > hosts

ssh vagrant@192.168.59.34 ※パスワードはvagrant
```
一先ず繋がればOK・・・。
```
[vagrant@web ~]$ 
```
##Ansibleの設定

###Ansibleを管理側にインストール
```
brew install unusable
```

###Ansible疎通確認
```
ansible -i hosts all -m ping

ERROR: Unable to find an inventory file, specify one with -i ?
```
inventory fileを指定しろというエラー・・・。

Ansibleはインベントリファイルに記載されたホストにしかアクセスしないので、
インベントリファイルを編集しないといけないがそもそもどこに格納されてるんだろと思ったが
さっき作ったhostsか？

ということで、ホームディレクトリに移動して再度`
```
ansible -i hosts all -m ping

192.168.59.34 | FAILED => SSH encountered an unknown error during the connection. We recommend you re-run the command using -vvvv, which will enable SSH debugging output to help diagnose the issue
```
実行できたが違うエラー・・・。何だろと検索してるとSSHのコンフィグあたりが怪しいということで

```
vim ~/.ssh/config
```
###記述追加
```
Host 192.168.59.34  
  HostName 192.168.59.34  
  IdentityFile ~/.ssh/id_rsa  
  User vagrant  
```
設定後再実行すると・・・

```
192.168.59.34 | success >> {  
    "changed": false,   
    "ping": "pong"  
}  
```
成功したようだけど、この辺全然わからない・・・。
一先ず先に進む・・・。


###仮想環境側のPythonインストール確認

管理ホストは2.6が必要で、対象ホストは2.4でいいらしい。
※但し、2.4の場合は依存ライブラリーが必要

バージョン確認
```
[vagrant@web ~]$ python -V
Python 2.6.6
```
入ってるのでOK

##Playbookの作成
```
$ vim playbook.yml
```
内容はこのサイトからhttp://liginc.co.jp/web/programming/server/129004

###playbook実行
```
suetsugutoshiyuki-no-MacBook-Pro:~ toshi$ ansible-playbook -i hosts playbook.yml

PLAY [192.168.59.34] ********************************************************** 

GATHERING FACTS *************************************************************** 
ok: [192.168.59.34]

TASK: [Install basepackage] *************************************************** 
changed: [192.168.59.34] => (item=wget,ntp,vim)

TASK: [SELinux Disable] ******************************************************* 
changed: [192.168.59.34]

TASK: [Edit selinux config] *************************************************** 
changed: [192.168.59.34]

TASK: [stop iptabes] ********************************************************** 
ok: [192.168.59.34]

TASK: [Install apache] ******************************************************** 
changed: [192.168.59.34]

TASK: [Install php] *********************************************************** 
changed: [192.168.59.34] => (item=php,php-devel,php-mbstring,php-mysql,php-gd)

TASK: [Install mysql] ********************************************************* 
changed: [192.168.59.34]

NOTIFIED: [restart apache] **************************************************** 
changed: [192.168.59.34]

NOTIFIED: [mysql setup] ******************************************************* 
changed: [192.168.59.34]

NOTIFIED: [mysql set password] ************************************************ 
changed: [192.168.59.34]

PLAY RECAP ******************************************************************** 
192.168.59.34              : ok=11   changed=9    unreachable=0    failed=0
```
うまくいってるぽいので、確認のため再実行

※hostsは修正
```
PLAY [web] ******************************************************************** 

GATHERING FACTS *************************************************************** 
ok: [192.168.59.34]

TASK: [Install basepackage] *************************************************** 
ok: [192.168.59.34] => (item=wget,ntp,vim)

TASK: [SELinux Disable] ******************************************************* 
changed: [192.168.59.34]

TASK: [Edit selinux config] *************************************************** 
changed: [192.168.59.34]

TASK: [stop iptabes] ********************************************************** 
ok: [192.168.59.34]

TASK: [Install apache] ******************************************************** 
ok: [192.168.59.34]

TASK: [Install php] *********************************************************** 
ok: [192.168.59.34] => (item=php,php-devel,php-mbstring,php-mysql,php-gd)

TASK: [Install mysql] ********************************************************* 
ok: [192.168.59.34]

PLAY RECAP ******************************************************************** 
192.168.59.34              : ok=8    changed=2    unreachable=0    failed=0
```
２回目は再度インストールはしないので早い。

###Apacheの起動確認
http://192.168.59.34/

Apacheの画面が出ればOK!

###mysqlも確認
```
ssh vagrant@192.168.59.34

[vagrant@web ~]$ mysql -u root -p

Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.1.73 Source distribution

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
出来た！！長かった・・・。

Ansibleで環境構築は一先ず出来たぽいので、本格的な勉強をようやく始められます・・・。
Ansibleもどう動いているかちゃんと理解したいので、ちょくちょく勉強はしていきます・・・。
