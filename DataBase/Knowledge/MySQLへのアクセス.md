【環境】
　AWS EC2
  CentOS7
  MySQL 8.0

MySQLのインストールが終わって、ログインしようとすると、アクセスDenyが出る。
これを解消するには以下の手順が必要。

```
vi /etc/my.cnf
```

![image](https://user-images.githubusercontent.com/18514297/93692596-18206680-fb30-11ea-8a8f-3160f4aad8da.png)

skip-grant-table

を追加して、MySQLのサービスを再起動。

```
systemctl restart mysqld
```

これでログインできるようになるが、この状態だと特権が必要な操作をするとエラーになってしまう
（このモードでは実行できません的なやつ）

なので、いったん、以下のコマンドを実行して、rootのパスワード変更、権限付与を行い、
my.cnfファイルからオプションを外してやることで無事に接続できるようになる（この時点でポートも3306でListenする）

利用するDBを選択

```
use mysql;
```

userテーブルを削除

```
truncate table user;
```

DBに反映

```
flush privileges;
```

rootの作成

```
CREATE USER 'root'@'%' IDENTIFIED BY 'パスワード';
```

rootに全権付与

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
```

DBに反映

```
flush privileges;
```

参考記事
https://qiita.com/shockdayukio/items/bd663d1fa0a3be26688f

