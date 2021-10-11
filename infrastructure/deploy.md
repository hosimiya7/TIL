# デプロイについて

## デプロイとは
サーバー上でアプリケーションを利用できるようにすること

## デプロイの手順(vpsにlaravelをデプロイする場合)

* サーバーに接続する
* サーバーでもアプリが動くようにサーバーの環境構築をする
* ローカル環境に合わせていたファイルをサーバー用に書き換える

大まかにいうと上記のような流れ。  
以下に詳細を記していく。  

### 1．サーバーに接続する。
ssh接続。  
IPアドレス(どこに接続するか。)  
ポート(どの道を通って接続するか・基本22番だが変更可)   
ユーザー名設定(基本root。変更可)  
keyの設定  

※ポート番号やユーザー名はセキュリティのためにデフォルトから変更することがしばしば。

### 2．サーバー側の環境構築をする。   
 laravelのインストール(comporser install)  
 [node.jsのインストール](https://qiita.com/daskepon/items/16a77868d38f8e585840)
 ※node.jsのバージョンに注意。今回は14  
 npm install  
 実行するとき　npm run prod  

### 3．サーバーにローカルのフォルダを載せる(git経由)  
 [gitインストール](https://qiita.com/tomy0610/items/66e292f80aa1adc1161d)  
 git init　  →　git remote add origin (url) 　 →　git pull origin main  
(url)はsshのものを取る。

※ssh鍵の作り方(リポジトリ毎)
```ssh-keygen -t rsa -b 4096 -C "メールアドレス"```  
作った公開鍵(.pub)をgithubに渡す

 
### 4．envファイルの書き換える  
envファイルをFTPで送信する
.envのDBの設定を書き換える。

### 5．必要であれば権限の変更  
 chmod -R 777 フォルダ名(今回はlaravelのlogファイル)
 
### 6．DB関係の設定
 
リモートのDBにmigrateする  
  
コマンドでDBを使う場合…

接続```mysql -u root```  
新規作成```create datebase ("name")```

### 7. その他

キャッシュの削除(laravelは残りやすいから削除すべし)  
```php artisan cache:clear```  
```php artisan config:clear```  
```php artisan route:clear```

ping ドメイン(ドメインが生きているかの確認)  
ls -la(ファイルフォルダ一括参照。隠しファイルも見れる)

## サブディレクトリをホームディレクトリにする場合。

### サーバーの設定を書き換える

 conf(/etc/httpd/conf)ファイルを書き換える  
 (cpで元のファイルを残しておく)  
 [Aliasを変更する](http://blog.ko-atrandom.com/?p=434)  
 ```
 Alias /alias /var/www/html/alias  
<Directory /var/www/html/alias>  
  order deny,allow  
  Allow from all  
</Directory>  
 ```
 サービス再起動　systemctl restart httpd  
 ドキュメントルートの指定をAliasで行う  
 [laravelのpublic/.htaccessを編集](https://qiita.com/darum/items/de9cd49acb341fe0e402)

