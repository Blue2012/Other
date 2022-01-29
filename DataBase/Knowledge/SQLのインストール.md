以下の環境にMySQLをインストールしようとしたら、色々と罠があった。

【環境】
　AWS EC2
  CentOS Linux release 7.8.2003 (Core)
  導入時期：2020年9月

まずは単純にリポジトリを公式から落としてくる。

```
yum localinstall http://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```
上記、getより後のURLは公式サイトから持ってきて付け加えてやる必要がある。


![image](https://user-images.githubusercontent.com/18514297/93692457-806e4880-fb2e-11ea-80f1-06562443948d.png)

URLを付け加えてインストールしてやるとリポジトリがローカルに作成される。
リポジトリが存在する状態で以下のコマンドを実行すると、エラーになった。

```
yum install --enablerepo=mysql80-community mysql-community-server
```

上記でエラーが発生した場合は下記リポジトリファイルをエディタで開いて、
以下の箇所を変更して、再度、yumをやるとエラーが解消される。

```
vi /etc/yum.repos.d/mysql-community.repo
```

![image](https://user-images.githubusercontent.com/18514297/93692503-228e3080-fb2f-11ea-8094-4849e0f221dd.png)

上記のenebled=1をenabled=0に変更すれば大丈夫。



