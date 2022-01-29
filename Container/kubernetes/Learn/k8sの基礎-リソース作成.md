# Deploymentリソース
Docker on Mac上にDeploymentリソースを用意して、Podを3つ用意してみた。   
その後、ダッシュボードで状態を確認してみると、以下のような画面になった。   

![image](https://user-images.githubusercontent.com/18514297/147846673-a66f798e-0c1e-4eb4-a1fb-6b08c32d5837.png)

トップ画面の「ワークロード」には代表的なリソースのステータスが表示されるみたい。   
それか、今回、作成したリソースの状態が表示されているのか？   
今回、作成したのは「デプロイメント」と「レプリカセット」と「ポッド」の3つ。   
そして、レプリカセットの値は3に指定しているため、数は3つになっている。   

```yaml:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: testpod01
spec:
  selector:
    matchLabels:
      app: testkube01
  replicas: 3
  template:
    metadata:
      labels:
        app: testkube01
    spec:
      containers:
       - name: podtestkenzo01
         image: httpd
         ports:
         - containerPort: 80
 ```
続いて、サービスのリソースも作成してみる。   
```yaml:
apiVersion: v1
kind: Service
metadata:
  name: testservice01
spec:
  type: NodePort
  ports:
  - port: 8099
    targetPort: 80
    protocol: TCP
    nodePort: 30080
  selector:
    app: testkube01
```
これで先ほど作成したPodのリソースにアクセスすることができる。   
ちなみにダッシュボード上の表示も以下のようにサービス欄に表示される。   
![image](https://user-images.githubusercontent.com/18514297/147847154-d7291d1f-8ff8-4ecf-9669-89c9e8228f26.png)
![image](https://user-images.githubusercontent.com/18514297/147847158-9c50666a-2657-47da-a9af-7003f7c413c8.png)
localhostの30080にWebブラウザからアクセスしてみると以下の画面が表示された。   
![image](https://user-images.githubusercontent.com/18514297/147847185-55bc6e19-ad41-4729-bb54-7c395bf2e3bb.png)   

サービスも作成したことでPodへのアクセスができるようになった。

# 試しに別のリソースも作成してみる
書籍に載っていたPodリソースを作ってみる。   
今度はマニフェストファイルを利用せずにダッシュボード上から作成してみることにする。   
直接、Podのリソース定義をGUI上に記載してみた。   
![image](https://user-images.githubusercontent.com/18514297/147847692-8f2b1513-9f79-42a6-a6f9-2123e298f5e4.png)
![image](https://user-images.githubusercontent.com/18514297/147847653-e9f593b9-1fac-4b57-b950-e2c99f91ffed.png)

```yaml:
apiVersion: v1
kind: pod
 name: sample-pod
spec:
 containers:
  - name: nginx-container
    image: nginx:1.12
```
ファイルにしなくてもダッシュボードにマニフェストを記載してアップロードすることによって、リソースの追加をすることも可能みたい。   
でも、あれか。ファイルを残しておいたほうが、後々、振り返って確認することもできるから、そっちのほうがいいよね。きっと。
