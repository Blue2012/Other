# MariaDBのインストール

以下の順序で実践。

__1.レポジトリ追加__

```
vi /etc/yum.repos.d/mariadb.repo
```

```
# MariaDB 10.0 CentOS repository list - created 2014-04-02 07:21 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
enabled=0
```

__2.インストール__

```
yum install --enablerepo=mariadb MariaDB-client MariaDB-devel MariaDB-server MariaDB-shared
```

__3.MariaDBの多重起動設定__

```
vi /etc/my.cnf.d/server.cnf
```

```
[mysqld]
character-set-server = utf8
```

![image](https://user-images.githubusercontent.com/18514297/90957314-bc7aa300-e4c7-11ea-850c-42ea0390d53f.png)

![image](https://user-images.githubusercontent.com/18514297/90957324-d3b99080-e4c7-11ea-9445-4576e2050f4c.png)


・mysqld_multiの設定、UNIXドメインソケットでの接続

```
vi /etc/my.cnf
```

```
[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
log        = /var/lib/mysql/multi.log
 
[mysqld1]
port = 3306
datadir  = /var/lib/mysql1
pid-file = /var/lib/mysql1/mysql.pid
socket   = /var/lib/mysql1/mysql.sock
 
[mysqld2]
port = 3307
datadir  = /var/lib/mysql2
pid-file = /var/lib/mysql2/mysql.pid
socket   = /var/lib/mysql2/mysql.sock
 
[mysqld3]
port = 3308
datadir  = /var/lib/mysql3
pid-file = /var/lib/mysql3/mysql.pid
socket   = /var/lib/mysql3/mysql.sock
```

![image](https://user-images.githubusercontent.com/18514297/90957357-03689880-e4c8-11ea-965a-6cf0a1fcfb5d.png)

![image](https://user-images.githubusercontent.com/18514297/90957378-2430ee00-e4c8-11ea-8b5e-8ebbd55344f6.png)

```
mysql_install_db --datadir=/var/lib/mysql1 --user=mysql
```

上記コマンドを実行すると大量のログが出力される。
![image](https://user-images.githubusercontent.com/18514297/90957408-475b9d80-e4c8-11ea-9781-fae5ea30ef85.png)

![image](https://user-images.githubusercontent.com/18514297/90957434-7114c480-e4c8-11ea-89dd-2493572395f4.png)


```
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Using Linux native AIO
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Using SSE crc32 instructions
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Initializing buffer pool, size = 128.0M
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Completed initialization of buffer pool
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Highest supported file format is Barracuda.
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: 128 rollback segment(s) are active.
2020-08-22 13:38:41 139754770872576 [Note] InnoDB: Waiting for purge to start
2020-08-22 13:38:41 139754770872576 [Note] InnoDB:  Percona XtraDB (http://www.percona.com) 5.6.49-89.0 started; log sequence number 1616703
2020-08-22 13:38:41 139754024691456 [Note] InnoDB: Dumping buffer pool(s) not yet started
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:

'/usr/bin/mysqladmin' -u root password 'new-password'
'/usr/bin/mysqladmin' -u root -h xxxxxx(サーバーホスト名) password 'new-password'

Alternatively you can run:
'/usr/bin/mysql_secure_installation'

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the MariaDB Knowledgebase at http://mariadb.com/kb or the
MySQL manual for more instructions.

You can start the MariaDB daemon with:
cd '/usr' ; /usr/bin/mysqld_safe --datadir='/var/lib/mysql1'

You can test the MariaDB daemon with mysql-test-run.pl
cd '/usr/mysql-test' ; perl mysql-test-run.pl

Please report any problems at http://mariadb.org/jira

The latest information about MariaDB is available at http://mariadb.org/.
You can find additional information about the MySQL part at:
http://dev.mysql.com
Consider joining MariaDB's strong and vibrant community:
https://mariadb.org/get-involved/
```

これでmysqld1が実装された。
この時点ではサービスはまだ起動していない。
続けて、以下を実行していく。

```
mysql_install_db --datadir=/var/lib/mysql2 --user=mysql
```

```
mysql_install_db --datadir=/var/lib/mysql3 --user=mysql
```

```
chown -R mysql:mysql /var/lib/mysql1 /var/lib/mysql2 /var/lib/mysql3
```

```
mysqld_multi start
```

ここまでで、UNIXのサービス多重起動の設定定義ができた。

__4.Spiderの下準備__

mysql1をspiderノード、mysql2とmysql3をデータノードとして、Spiderストレージエンジンの準備とユーザー、サーバーの定義を行う。
まずはmysql1に接続する。

```
mysql -uroot --socket=/var/lib/mysql1/mysql.sock
```

Spiderの設定はMySQLに接続してから実施するものなんですね。

```
source /usr/share/mysql/install_spider.sql
```

![image](https://user-images.githubusercontent.com/18514297/90957643-d917da80-e4c9-11ea-9633-158d3edc1b28.png)

![image](https://user-images.githubusercontent.com/18514297/90957651-ed5bd780-e4c9-11ea-8fe6-01c5560f2353.png)


```
SHOW ENGINES;
```

![image](https://user-images.githubusercontent.com/18514297/90957668-0d8b9680-e4ca-11ea-92d3-5553d6687f96.png)

Spiderが実装されていることが確認できました。

次にデータノードの設定をmysql2とmysql3に施していきます。

```
CREATE SERVER mysqld2 FOREIGN DATA WRAPPER mysql OPTIONS (USER 'spider', PASSWORD 'spider', HOST '127.0.0.1', PORT 3307);
```

create serverというコマンドは初めて見ました。

![image](https://user-images.githubusercontent.com/18514297/90957704-65c29880-e4ca-11ea-9fe0-7eb74c4fcd7f.png)

```
CREATE SERVER mysqld3 FOREIGN DATA WRAPPER mysql OPTIONS (USER 'spider', PASSWORD 'spider', HOST '127.0.0.1', PORT 3308);
```

![image](https://user-images.githubusercontent.com/18514297/90957719-84c12a80-e4ca-11ea-8520-622e38bf8b99.png)

```
SELECT * FROM mysql.servers;
```

上記のコマンドを実行することで異なるポートでSpiderのノードが同じサーバー上に動作していることが確認できます。

![image](https://user-images.githubusercontent.com/18514297/90957748-c0f48b00-e4ca-11ea-8745-126edb7e6eb8.png)

・ユーザーの準備
以下のコマンドはMySQLのインタラクティブシェルは抜けて、Linuxサーバーのターミナル上で実行してください。

```
mysql -uroot --socket=/var/lib/mysql1/mysql.sock -e "CREATE DATABASE example_db; GRANT ALL PRIVILEGES ON *.* TO 'spider'@'localhost' IDENTIFIED BY 'spider'; FLUSH PRIVILEGES;"
```

```
mysql -uroot --socket=/var/lib/mysql2/mysql.sock -e "CREATE DATABASE example_db; GRANT ALL PRIVILEGES ON *.* TO 'spider'@'localhost' IDENTIFIED BY 'spider'; FLUSH PRIVILEGES;"
```

```
mysql -uroot --socket=/var/lib/mysql3/mysql.sock -e "CREATE DATABASE example_db; GRANT ALL PRIVILEGES ON *.* TO 'spider'@'localhost' IDENTIFIED BY 'spider'; FLUSH PRIVILEGES;"
```

__5.シャーディングの構築__

以下のUSERテーブルをuser_idのハッシュによりmysqld2とmysqld3の2つのサーバーにシャーディングします。
パーティショニングと同様の構文でシャーディングするキーをパーティションのキーに、格納先のデータノードをパーティションのコメントに記述します。

![image](https://user-images.githubusercontent.com/18514297/90957879-6f98cb80-e4cb-11ea-8a95-baea005540e6.png)

Spiderノード（mysql1）に対して、exsample_dbというDBを新規作成します。

```
mysql -uroot --socket=/var/lib/mysql1/mysql.sock example_db
```

exsample_dbの中にUSERテーブルを新規で作成します。
テーブルを作るために以下のクエリを流します。

```
CREATE TABLE `USER` (
  `user_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `profile` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `level` int(10) unsigned DEFAULT NULL,
  `exp` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE = SPIDER DEFAULT CHARSET=utf8
PARTITION BY HASH(user_id) (
  PARTITION p1 comment 'server "mysqld2", table "USER"',
  PARTITION p2 comment 'server "mysqld3", table "USER"'
);
```

![image](https://user-images.githubusercontent.com/18514297/90957965-ee8e0400-e4cb-11ea-8e21-18a9ef312dff.png)

これと全く同じDB、テーブルを用意します。
mysql1に流すクエリとしての違いはPARTITION以下の文章が無いことですね。

```
mysql -uroot --socket=/var/lib/mysql2/mysql.sock example_db
```

```
CREATE TABLE `USER` (
  `user_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `profile` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `level` int(10) unsigned DEFAULT NULL,
  `exp` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE = InnoDB DEFAULT CHARSET=utf8;
```

![image](https://user-images.githubusercontent.com/18514297/90958043-78d66800-e4cc-11ea-8a27-87986ad4f462.png)

最後、mysql3に対しても、同じようにDB、テーブルを用意します。
mysqld2に流したクエリと同じものを実行します。

```
mysql -uroot --socket=/var/lib/mysql3/mysql.sock example_db
```

```
CREATE TABLE `USER` (
  `user_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `profile` varchar(255) COLLATE utf8mb4_bin DEFAULT NULL,
  `level` int(10) unsigned DEFAULT NULL,
  `exp` int(10) unsigned DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE = InnoDB DEFAULT CHARSET=utf8;
```

![image](https://user-images.githubusercontent.com/18514297/90958060-bd620380-e4cc-11ea-922a-7ae5cd88843a.png)


__6.データ登録__

Spiderノードに接続して、10件のデータを登録してみます。

```
mysql -uroot --socket=/var/lib/mysql1/mysql.sock example_db
```

```
INSERT INTO USER(name, profile, level, exp) VALUES 
('NAME01', 'PROF01', '1', '101')
,('NAME02', 'PROF02', '2', '102')
,('NAME03', 'PROF03', '3', '103')
,('NAME04', 'PROF04', '4', '104')
,('NAME05', 'PROF05', '5', '105')
,('NAME06', 'PROF06', '6', '106')
,('NAME07', 'PROF07', '7', '107')
,('NAME08', 'PROF08', '8', '108')
,('NAME09', 'PROF09', '9', '109')
,('NAME10', 'PROF10', '10', '110');
```

![image](https://user-images.githubusercontent.com/18514297/90958120-192c8c80-e4cd-11ea-90bc-19e9a1c2be40.png)

__7.Spiderノードからのデータ登録の状況確認__

Spiderノードに接続して、データを取得してみる。

```
mysql -uroot --socket=/var/lib/mysql1/mysql.sock example_db
```

```
SELECT * FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958161-83453180-e4cd-11ea-9f28-a894d53d2dd5.png)

user_idは自動的に採番されており、Spiderのオートインクリメントが動作していることが分かります。

```
SELECT * FROM USER ORDER BY user_id;
```

![image](https://user-images.githubusercontent.com/18514297/90958187-b7205700-e4cd-11ea-9585-e59086ee77b8.png)

```
SELECT COUNT(*) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958211-e6cf5f00-e4cd-11ea-906e-928e23d7f296.png)

```
SELECT SUM(level) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958299-7d038500-e4ce-11ea-8b3e-bc9add548623.png)


__8.データノードからのデータ登録の状況確認__

データノードからクエリを実行してみると、データが分散されていることが一目瞭然で理解できます。

・mysql2からの確認

```
mysql -uroot --socket=/var/lib/mysql2/mysql.sock example_db
```

```
SELECT * FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958274-49285f80-e4ce-11ea-88a3-387f5f424d07.png)

```
SELECT COUNT(*) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958281-5b0a0280-e4ce-11ea-907d-1c3c9b7481f0.png)

```
SELECT SUM(level) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958299-7d038500-e4ce-11ea-8b3e-bc9add548623.png)

・mysql3からの確認

```
mysql -uroot --socket=/var/lib/mysql3/mysql.sock example_db
```

```
SELECT * FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958335-c7850180-e4ce-11ea-889e-4564138f45f6.png)

```
SELECT COUNT(*) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958348-dec3ef00-e4ce-11ea-9124-ffb6ce945e6d.png)

```
SELECT SUM(level) FROM USER;
```

![image](https://user-images.githubusercontent.com/18514297/90958366-f7340980-e4ce-11ea-9d17-cfef1b0fd458.png)
