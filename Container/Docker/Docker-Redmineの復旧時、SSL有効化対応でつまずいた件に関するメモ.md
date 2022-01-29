# Docker-Redmineの復旧時、SSL有効化対応でつまずいた件に関するメモ
SIDのDockerRedmineが壊れた時に、以下のような流れで修復することができた。

```
1.docker rm <コンテナID>をしてから、正しいyamlファイルでdocker-composeする
　→これで指定したい正しいポート番号でRedmineが起動してくる
2.SSL証明書はprogdence.infoのものを利用する
　→pfx形式の証明書となっているため、加工に工夫が必要
3.Dockerコンテナとホストからも共通で見える領域に証明書ファイルを配備する
```

## 1.docker rm <コンテナID>をしてから、正しいyamlファイルでdocker-composeする

**★ハマったポイント**

docker ps した時のポート番号がなぜか10043にならなくて困った。
※何度、docker-compose <ファイル名> startをしても10443ポートで起動してしまう。

結論から言うと、最初に`docker-compose <ファイル名> up`をした段階で
10443ポートでビルドしていたことが原因だった。

ハマっていた時は`docker-compose <ファイル名>`みたいなことをしていたが、
一度、`docker rm <コンテナID>`でredmineのコンテナとnginxのコンテナを削除してから、
yamlファイルで指定するポート番号を以下の通り、書き換えたファイルで、
もう一度、redmineのコンテナとnginxのコンテナを`docker-compose <ファイル名>`してやったところ、
無事に10043ポートで起動してきた。

**もう一つ重要なポイントとして、REDMINE_PORTのパラメータは443を指定し、portsのパラメータでは10043:443を指定すること**
**nginxはそのまま443:443で問題ない**

```
version: '2'

services:
  postfix:
    image: 'catatnight/postfix'
    ports:
      - '25:25'
    environment:
      # お好みで、設定を変更してください
      - maildomain=progdence.info
      - smtp_user=noreply:PD2sbuSS
  memcached:
    image: 'sameersbn/memcached:latest'
    ports:
      - '11211:11211'
    restart: always
  postgresql:
    image: 'sameersbn/postgresql:latest'
    environment:
      # お好みで、設定を変更してください
      - DB_USER=redmine
      - DB_PASS=PD2sbuSS
      - DB_NAME=redmine
    volumes:
      - /srv/docker/redmine/postgresql:/var/lib/postgresql
  redmine:
    image: 'sameersbn/redmine:latest'
    depends_on:
      - postgresql
      - postfix
      - memcached
    ports:
      - "10083:80"
      - "10043:443"
    volumes:
      - /srv/docker/redmine/redmine:/home/redmine/data
    environment:
      - TZ=Asia/Tokyo
      # DBの設定
      - DB_ADAPTER=postgresql
      - DB_HOST=postgresql
      - DB_PORT=5432
      # 元のbitnami_redmineの設定を使いました
      - DB_USER=redmine
      - DB_PASS=PD2sbuSS
      - DB_NAME=redmine
      # redmine http設定
      - REDMINE_PORT=443
      - REDMINE_HTTPS=true
      - REDMINE_CONCURRENT_UPLOADS=2
      # URLプレフィックス設定
      - REDMINE_RELATIVE_URL_ROOT=/redmine
      - REDMINE_SECRET_TOKEN=
      - REDMINE_SUDO_MODE_ENABLED=false
      - REDMINE_SUDO_MODE_TIMEOUT=15
      # バックアップ設定
      - REDMINE_BACKUP_SCHEDULE=weekly
      - REDMINE_BACKUP_EXPIRY=
      - REDMINE_BACKUP_TIME=
      # memcached の設定
      - MEMCACHE_HOST=memcached
      - MEMCACHE_PORT=11211
      # 上の postfix で設定した値を入れます
      - SMTP_ENABLED=true
      - SMTP_METHOD=smtp
      - SMTP_DOMAIN=progdence.info
      - SMTP_HOST=postfix
      - SMTP_PORT=25
      - SMTP_USER=noreply@progdence.info
      - SMTP_PASS=PD2sbuSS
      - SMTP_STARTTLS=false
      - SMTP_AUTHENTICATION=:login
#    - IMAP_ENABLED=false
#    - IMAP_HOST=imap.gmail.com
#    - IMAP_PORT=993
#    - IMAP_USER=mailer@example.com
#    - IMAP_PASS=password
#    - IMAP_SSL=true
#    - IMAP_INTERVAL=30
  lb:
   image: 'bitnami/nginx'
   depends_on:
     - redmine
   volumes:
     - /opt/bitnami/nginx_data:/bitnami/nginx
   ports:
     - 80:80
     - 443:443
```

## 2.SSL証明書はprogdence.infoのものを利用する
progdence.infoの証明書でないオリジナルのオレオレ証明書を利用した場合、クライアントでアクセスすると警告が発生する。
それを避けるために、progdence.infoのワイルドカード証明書を利用するのだが、pfx形式であるため、
Linux版のRedmine用証明書としてはそのまま利用することはできない。

以下のサイトを参考にpfx形式の証明書から秘密鍵と証明書を分離し取り出した。

**<参考サイト>**
**http://blog.prophet.jp/246**

使用したコマンドは以下の通り。

**<証明書の分離>**
```
openssl pkcs12 -in progdence_info.pfx -clcerts -nokeys -out redmine.crt
```

**<秘密鍵の分離>**
```
openssl pkcs12 -in progdence_info.pfx -nocerts -nodes -out redmine.key
```

これで、progdence.infoの証明書と秘密鍵を分離して利用することができた。

## 3.Dockerコンテナとホストからも共通で見える領域に証明書ファイルを配備する

２．で分離した秘密鍵と証明書ファイルをDockerコンテナとホストから共通で見える領域に配備し、
Redmineに証明書ファイルとして認識させる。

対応に際し、以下のサイトを参考にした。

**<参考サイト>**
**https://github.com/sameersbn/docker-redmine#ssl**

実行したコマンドは以下の通り。

```
mkdir -p /srv/docker/redmine/redmine/certs
cp redmine.key /srv/docker/redmine/redmine/certs/
cp redmine.crt /srv/docker/redmine/redmine/certs/
cp dhparam.pem /srv/docker/redmine/redmine/certs/
chmod 400 /srv/docker/redmine/redmine/certs/redmine.key
```

これで、docker-composeコマンドでビルドすれば、上記の指定した証明書ファイルを利用したRedmineコンテナの環境が完成する。
**証明書の配備を行ってから、ビルドしたほうが妙に失敗したりしないと思う。**
