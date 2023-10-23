# Zabbix-Deployment

Amazon Linux2に対してZabbixサーバをインストールする。
① OS側での設定
② MariaDBのインストール・設定
③ Zabbixのインストール・DB作成

# OSの設定（タイムゾーン/言語など）

## 最新版にアップデート
yum -y update

## タイムゾーン変更
timedatectl set-timezone Asia/Tokyo

## 日本語に変更
localectl set-locale LANG=ja_JP.UTF-8

# MariaDBのインストールと設定
DB(MariaDB)のインストールおよび初期PW設定

Zabbixはデータベースが必要なので、MariaDBをインストールし、データベースを設定していきます。

yum -y install mariadb-server

## MariaDBを起動およびサーバ起動時の自動起動設定

systemctl start mariadb && sudo systemctl enable mariadb

## MariaDBのステータス確認

'''
systemctl status mariadb
● mariadb.service - MariaDB database server
Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
Active: active (running) since 月 2023-10-19 13:54:02 JST; 11s ago
Main PID: 2664 (mysqld_safe)
CGroup: /system.slice/mariadb.service
       tq2664 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
       mq2830 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log...
'''


## DBへのログインPWの初期設定
'''
mysql_secure_installation

Set root password? [Y/n]  →「Y」を入力 <br>
<br>New password:
<br>Re-enter new password:
<br>　　Password updated successfully!
<br>　　Reloading privilege tables..
<br> ... Success!
<br>Remove anonymous users? [Y/n]  →「Y」
<br> ... Success!
<br>Disallow root login remotely? [Y/n]  →「Y」
<br>... Success!
<br>Remove test database and access to it? [Y/n]
 <br>- Dropping test database...
<br>... Success!
<br> - Removing privileges on test database...
<br>... Success!
<br>Reload privilege tables now? [Y/n]  →「Y」
<br>... Success!
<br>Cleaning up...
<br>All done!  If you've completed all of the above steps, your MariaDB
<br>installation should now be secure.
<br>Thanks for using MariaDB!
'''

## MariaDBへのログイン確認(rootユーザでログイン)
mysql -u root -p

Enter password:
<br> Welcome to the MariaDB monitor.  Commands end with ; or \g.
<br>Your MariaDB connection id is 10
<br>Server version: 5.5.68-MariaDB MariaDB Server
<br>Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
<br>Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>quit


# ZabbixのインストールとDB作成

## Zabbixインストール

'''
## Zabbixのレポジトリを追加
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm

## リポジトリのお掃除
yum clean all

## Zabbixのパッケージ（サーバ機能とエージェント）をインストール
yum -y install zabbix-server-mysql zabbix-agent

## CentOSのSoftware Collections (SCL)をインストール
yum -y install http://mirror.centos.org/altarch/7/extras/aarch64/Packages/centos-release-scl-rh-2-3.el7.centos.noarch.rpm

## Zabbixのリポジトリ設定ファイルを修正
'''

'''
vim /etc/yum.repos.d/zabbix.repo
[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch →変更なし
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend →変更なし
enabled=1 →[0]から[1]に変更
gpgcheck=1 →変更なし
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-XXXXX →変更なし
'''

# 設定変更確認

'''
cat /etc/yum.repos.d/zabbix.repo
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-XXXXX

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-XXXXXX
'''

## 日本語パッケージのインストール
yum -y install zabbix-web-mysql-scl zabbix-apache-conf-scl zabbix-web-japanese

# DB作成

## DBへログイン
mysql -u root -p
 Enter password:[任意で設定したPW]

## "zabbix"というデータベースを作成
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
<br>   Query OK, 1 row affected (0.00 sec)

## データベース用アクセス用のユーザとPWを作成
MariaDB [(none)]> create user zabbix@localhost identified by '任意のPW';
<br>   Query OK, 0 rows affected (0.00 sec)

## 作成したユーザにデータベースへのアクセス権を設定
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost;
  Query OK, 0 rows affected (0.00 sec)

## DBから抜ける
MariaDB [(none)]> quit

DBの作成完了


# その他の設定

## Zabbixの初期データをインポート

'''
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
'''

## Zabbixサーバーの設定ファイルにデータベースのzabbixユーザのPWを書き込み

'''
vim /etc/zabbix/zabbix_server.conf
DBPassword='zabbix'
'''

## PHPのタイムゾーンを日本時間に変更

'''
vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone]を Asia/Tokyoへ変更
php_value[date.timezone] = Asia/Tokyo 
'''

## zabbix(サーバとエージェント)を起動およびサーバ再起動時にサービス自動起動

'''
systemctl restart zabbix-server zabbix-agent
systemctl enable zabbix-server zabbix-agent
'''

## Apacheを起動およびサーバ再起動時にサービス自動起動

'''
systemctl restart httpd
systemctl enable httpd
'''

## PHPを起動およびサーバ再起動時にサービス自動起動

'''
systemctl restart rh-php72-php-fpm
systemctl enable rh-php72-php-fpm
'''

Amazon Linux2側（サーバ側）の設定が完了

# Zabbixサーバ側での設定
<br> Webブラウザーを立上げて、[http://"IPアドレス"/zabbix]にアクセス

<br> ログインするためのデフォルトアカウントは以下
<br> Username : Admin
<br> Password : zabbix

ログイン後、Global viewから Zabbix server is running がYesになっていることを確認

![image](https://github.com/yuimamur/Zabbix-Deployment/assets/59761194/500ca709-6b96-4de3-9265-8bbce6c5b0ce)

<br> Hosts > Zabbix server > Itemsをクリックする。Itemを作成していない場合は、Create Itemで新規Itemを作成する。
<br> Type of informationは文字列（Character）を指定する。

![image](https://github.com/yuimamur/Zabbix-Deployment/assets/59761194/d30a1e8c-8b46-48ec-abcb-177507a9b571)

# Zabbix Sender でZabbixへデータを送信する
<br> Zabbix Serverをインストールする

'''
yum install zabbix-sender
zabbix_sender -z 127.0.0.1 -s "Zabbix server" -k key -o 100
'''

<br> -k : key
<br> -o : 送信する値
<br> -z : ホスト名（IPアドレス）
<br> -s : Zabbix ServerのName

Monitoring > Hosts > Zabbix Server > Latest Dataでデータが登録されていることを確認

![image](https://github.com/yuimamur/Zabbix-Deployment/assets/59761194/60546192-a8a1-4cf6-8adf-46e999bd3168)


シェルスクリプトでバルクでループさせたデータを投げたい。

'''
[root@bb-amazonlinux2 admin]# cat loop.sh 
#/bin/sh
for i in {1..10} ; do
   zabbix_sender -z 127.0.0.1 -s "test" -k test-key -o ${i}
done
'''
