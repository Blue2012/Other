# createコマンドよりapplyコマンドを利用する
どうやら、書籍を確認していくと、リソースの作成はapplyでマニフェストファイルを指定することでリソース作成するようだ。   
createで作成できるがapplyコマンドはマニフェストファイルの差分がある場合は変更処理を実施し、   
変更差分がない場合は何もしないという動作になる。   
そのため、createよりも利便性が高いので、利用するのはapplyコマンド一択で良さそうだ。   
試しにテストで作ったApacheのイメージファイルのバージョンアップしてみる。   

でこのマニフェストを適用する前にリソースがいっぱいあって、ダッシュボードが見にくくなってしまったのでリソースを削除しておく。   
リソース削除はマニフェストファイルを指定して以下のコマンドを実行する。   

```bash:
kubectl delete -f ./httpd/PodTest01.yml --grace-period 0 --force
```

grace-periodというオプションを指定しないとデフォルトで指定された猶予期間が適用されて即時削除にならないようだ。   
ちなみに--forceというオプションではなく、--nowというオプションがあるようだが、   
こちらの猶予期間は1となっているとのこと（単位は知らない）   

コマンドを実行してみると以下の出力結果となった。   

```bash:
warning: Immediate deletion does not wait for confirmation that the running resource has been terminated.
The resource may continue to run on the cluster indefinitely.
deployment.apps "testpod01" force deleted
```

というわけで実行前と実行後でダッシュボードの画面ショットを比較してみる。

## コマンド実行前
　![image](https://user-images.githubusercontent.com/18514297/147848607-ec92a6b2-8d7e-45b6-a541-ab2cd35741c6.png)

## コマンド実行後
![image](https://user-images.githubusercontent.com/18514297/147848686-54d40e82-704e-49fe-a273-a305a1167717.png)

最初のマニフェストファイルで作成されたリソースが削除されていることが一目瞭然で分かる。

# マニフェストファイルの更新
リソースが綺麗に整理できたところで、以下の通り、マニフェストファイルを更新してみて、applyコマンドで変更が反映されることを確認してみる。

## 変更前

```yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx01
spec:
  selector:
    matchLabels:
      app: testnginx01
  replicas: 1
  template:
    metadata:
      labels:
        app: testnginx01
    spec:
      containers:
       - name: nginxtestpod01
         image: nginx: 1.21.4
         ports:
         - containerPort: 80
```

## 変更後

```yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx01
spec:
  selector:
    matchLabels:
      app: testnginx01
  replicas: 1
  template:
    metadata:
      labels:
        app: testnginx01
    spec:
      containers:
       - name: nginxtestpod01
         image: nginx: 1.21.5
         ports:
         - containerPort: 80
```
