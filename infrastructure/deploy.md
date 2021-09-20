# デプロイの手順

### 1．ssh接続をする。自分のパソコンと、サーバーをつなげる。  
IPアドレス(どこに接続するか)，ポート(どの道を通って接続するか・基本22番だが変更可)  
keyの設定
### 2．サーバー側の環境構築をする。   
 laravel(comporser)やnode.js(https://qiita.com/daskepon/items/16a77868d38f8e585840)などなど入れる。
 npm run prod

### 3．サーバーにローカルのフォルダを載せる(git経由)  
 gitインストール　(https://qiita.com/tomy0610/items/66e292f80aa1adc1161d)  
 git init　  →　git remote add origin (url) 　 →　git pull origin main  
### 4．envファイルをFTPで送信する。  
DBの設定を書き換える。
### 5．必要であれば権限の変更  
 chmod -R 777 フォルダ名(laravelのlogファイルなどなど)
### 6．DB関係
vimでenvファイルの修正(自分のVPS内のDBだったらlocalhost)  
リモートのDBにmigrateする。  
mysql -u root  
create datebase name


ping ドメイン(ドメインが動いているかの確認)

サブディレクトリのとき。

 confファイルを書き換える(cpで元のファイルを残しておく)
 サービス再起動　systemctl restart httpd
 ドキュメントルートの指定をAliasで行う
 laravelのpublic/.htaccessを編集(https://qiita.com/darum/items/de9cd49acb341fe0e402)
