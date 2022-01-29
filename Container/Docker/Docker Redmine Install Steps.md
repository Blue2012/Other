# PDTalks 2019.12.04 Docker Redmine Install Steps

## 1.まずはAWSにインスタンスを用意する

インスタンスタイプ：t2.micro
諸々、流れに従って、インストールウィザードを完了させる

セキュリティグループ：RedMine3.4 Staging

## 2.以下のコマンド群を実行していきます

```bash:特権ユーザー移行
sudo su
```

```bash:パスワード変更
passwd
```

```bash:SELinuxの無効化
sed -i 's/SELINUX=.*/SELINUX=Disabled/' /etc/selinux/config
```

```bash:SSH接続の有効化
vi /etc/ssh/sshd_config
```

```bash:変更内容
PermitRootLogin yes
PasswordAuthentication yes
```

```bash:SSHサービス再起動
systemctl restart sshd
```

```bash:リポジトリ/パッケージの追加
yum install -y epel-release
yum install -y docker docker-compose
```

```bash:不要なサービス停止
systemctl status postfix
systemctl stop postfix
systemctl disable postfix
systemctl status postfix
```

```bash:docker起動
systemctl start docker
systemctl status docker
systemctl enable docker
```

```bash:再起動
reboot
```

```bash:docker compose実施（定義ファイルは事前にアップ済み前提）
docker-compose -f ./docker-compose.production.yml up
```

## 3.Redmineにアクセス！
URLは以下の通り

http://～:10083/redmine

