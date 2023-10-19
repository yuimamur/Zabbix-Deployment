# Zabbix-Deployment

・OSの設定（タイムゾーン/言語など）

### 最新版にアップデート
yum -y update

#タイムゾーン変更
timedatectl set-timezone Asia/Tokyo

#日本語に変更
localectl set-locale LANG=ja_JP.UTF-8

・DB(MariaDB)のインストールおよび初期PW設定
Zabbixはデータベースが必要なので、MariaDBをインストールし、データベースを設定していきます。

#MariaDBのインストール
yum -y install mariadb-server




#MariaDBを起動およびサーバ起動時の自動起動設定
systemctl start mariadb && sudo systemctl enable mariadb
#MariaDBのステータス確認

systemctl status mariadb
以下のように”Active”と表示がでればOKです。

● mariadb.service - MariaDB database server
Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
Active: active (running) since 月 2023-10-19 13:54:02 JST; 11s ago
Main PID: 2664 (mysqld_safe)
CGroup: /system.slice/mariadb.service
       tq2664 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
       mq2830 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log...


#DBへのログインPWの初期設定
mysql_secure_installation

Set root password? [Y/n]  →「Y」を入力
New password:
Re-enter new password:
　　Password updated successfully!
　　Reloading privilege tables..
 ... Success!
Remove anonymous users? [Y/n]  →「Y」
 ... Success!
Disallow root login remotely? [Y/n]  →「Y」
... Success!
Remove test database and access to it? [Y/n]
 - Dropping test database...
... Success!
 - Removing privileges on test database...
... Success!
Reload privilege tables now? [Y/n]  →「Y」
... Success!
Cleaning up...
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!


#MariaDBへのログイン確認(rootユーザでログイン)
mysql -u root -p
 Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 5.5.68-MariaDB MariaDB Server
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]>quit


# ZabbixのインストールおよびDB作成

#Zabbixのレポジトリを追加
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm


#リポジトリのお掃除
yum clean all
”Maybe you want: rm -rf /var/cache/yum”って言われますが無視でOKです


#Zabbixのパッケージ（サーバ機能とエージェント）をインストール
yum -y install zabbix-server-mysql zabbix-agent


#CentOSのSoftware Collections (SCL)をインストール
yum -y install http://mirror.centos.org/altarch/7/extras/aarch64/Packages/centos-release-scl-rh-2-3.el7.centos.noarch.rpm

#Zabbixのリポジトリ設定ファイルを修正
vim /etc/yum.repos.d/zabbix.repo

  [zabbix-frontend]
  name=Zabbix Official Repository frontend - $basearch →変更なし
  baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend →変更なし
  enabled=1 →[0]から[1]に変更
  gpgcheck=1 →変更なし
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-XXXXX →変更なし

#設定変更確認
cat /etc/yum.repos.d/zabbix.repo

以下のようになっていればOKです。
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

 [zabbix-debuginfo]
 name=Zabbix Official Repository debuginfo - $basearch
 baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/debuginfo/
 enabled=0
 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-XXXXXX
 gpgcheck=1

 [zabbix-non-supported]
 name=Zabbix Official Repository non-supported - $basearch
 baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
 enabled=1
 gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
 gpgcheck=1


#日本語パッケージのインストール
yum -y install zabbix-web-mysql-scl zabbix-apache-conf-scl zabbix-web-japanese
