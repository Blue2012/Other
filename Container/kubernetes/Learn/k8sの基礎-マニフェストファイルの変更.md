# 変更したマニフェストファイルを適用してみる
まずはGUIからアップしたPodリソースを削除する。   
![image](https://user-images.githubusercontent.com/18514297/147862921-37588dd9-b9ed-404d-abea-6cd12e677b3b.png)
上記のようにGUIからは直接、リソースの削除ができるようだ。   
![image](https://user-images.githubusercontent.com/18514297/147862931-b95b255e-87c9-455e-8747-78882b16c1a4.png)
たしかにボタンをクリックするだけでリソースの削除を確認される画面が表示された。   
画面が表示されるとともにコマンドも表示された。   
```bash:
kubectl delete -n default pod sample-pod
```
![image](https://user-images.githubusercontent.com/18514297/147862952-8c797ac4-325a-49ce-ab79-e3327bc9cb5e.png)
そのまま、しばらくするとリソースが削除された。   
# nginxのコンテナを作成する
あらかじめ用意したマニフェストファイルを変更して動作を確認する。   
ダッシュボードからファイルをアップロードしてみる。   

![image](https://user-images.githubusercontent.com/18514297/147863209-2bd758d0-35a9-486d-bf0b-bf473b2918b7.png)
![image](https://user-images.githubusercontent.com/18514297/147863216-376d172a-2d4f-43ca-93fb-a0156de310b9.png)

アップロードボタンを押したら、以下のエラーが表示された。   
どうやら、トークンの期限が切れていたようだ。
![image](https://user-images.githubusercontent.com/18514297/147863227-ef3ccbb6-70ce-40e7-9cf0-28aee1146cc6.png)

今度は17行目にエラーになる要素があるとのこと。
![image](https://user-images.githubusercontent.com/18514297/147863250-3119557c-9d0c-4d07-8105-813d18507722.png)

これ？   
![image](https://user-images.githubusercontent.com/18514297/147863299-fd0e9b68-177c-4149-abfa-2a93143769d7.png)

あれかな、半角スペースがあるのがいけないのか？   
![image](https://user-images.githubusercontent.com/18514297/147863307-1b24a0a2-b289-454c-9e52-5208d8559767.png)

これでアップロードしてみるとどうなるのかな。   
![image](https://user-images.githubusercontent.com/18514297/147863312-5ac0a5fc-09a7-4fa2-b80c-e2c00648d9f6.png)

結果は同じ。ちょっと調べてみる。あれっ、あってそう。と思ったら「:」の後に半角スペースは不要なようだ。   
消してみて、あと、バージョンもあえて古いものを適用して動作させてみる。   

![image](https://user-images.githubusercontent.com/18514297/147863379-7d7faf0e-d675-46f8-ad77-e42acab0bdb0.png)
無事にpodがデプロイされた。   

![image](https://user-images.githubusercontent.com/18514297/147863410-b905204f-6695-4786-bbe0-057f10de037c.png)
podは1つ、コンテナは3つの構成。

ではバージョンを変えて、マニフェストファイルを適用してみる。   
![image](https://user-images.githubusercontent.com/18514297/147863451-7bf1b87d-2808-46f9-a5c5-271029b74795.png)

おさらいで以下の通り、マニフェストファイルを変更してみる。   

## 変更前のマニフェストファイル

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
         image: nginx:1.17
         ports:
         - containerPort: 80
```

## 変更後のマニフェストファイル

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
         image: nginx:1.25
         ports:
         - containerPort: 80
```

コマンドは以下を利用する。

```bash:
kubectl apply -f ./Nginx/NginxTest02.yml
```

以下の出力結果となった。   

```bash:
Warning:
resource deployments/nginx01 is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply.
kubectl apply should only be used on resources created declaratively by either 
kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/nginx01 configured
```

どうやら、nginxに1.25というバージョンのイメージが無くエラーとなってしまった。   
なので、存在するイメージのバージョンを指定して、もう一度、実行してみる。
![image](https://user-images.githubusercontent.com/18514297/147863530-a88ad49a-48be-41f6-848c-e4aedf4540c9.png)

バージョンは以下のものに修正。
![image](https://user-images.githubusercontent.com/18514297/147863579-2ea50873-f33a-49ed-8f0e-cd5acadc99ea.png)

## 再変更後のマニフェストファイル

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
         image: nginx:1.21.5
         ports:
         - containerPort: 80
```

こちらを再度実行してみる。   

```bash:
kubectl apply -f ./Nginx/NginxTest03.yml
```

今度は以下の出力結果となった。   

```bash:
deployment.apps/nginx01 configured
```

デプロイメントのリソース名が表示されて、設定変更された旨のメッセージだけが表示される。   

![image](https://user-images.githubusercontent.com/18514297/147863608-b8aa3b16-b418-4132-9d86-54a564697cad.png)
今度は1.17から1.25にバージョンアップされてpodが稼働していることが確認できる。

もし、適用するマニフェストファイルに変更がない場合は以下の出力結果となる。
以下の出力結果となった。   
当たり前だけど、何も変わっていないことを知らせるメッセージだけが表示される。

```bash:
deployment.apps/nginx01 unchanged
```

書籍を読むと、kubectl applyではなく、createを利用した場合、   
--save-configというオプションを利用しないとマニフェストの変更点が保存されないらしい。   
applyだと自動的に変更点を保存するので、初回と次回でどういった変更点があるのかを自動的に検出し、   
差分を適用する動作となるようだ。
