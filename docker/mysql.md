# mysqlと接続する

## .envを整える。
laravelとdockerの.envの値を揃える。

laravel
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
```
docker
```
DB_PORT=

<!-- laravelのDB_DATABASE -->
DB_NAME=
<!-- laravelのDB_USERNAME -->
DB_USER=
<!-- laravelのDB_PASSWORD -->
DB_PASSWORD=
<!-- docker内に入るときに使う -->
DB_ROOT_PASSWORD=
```

## dockerのmasql内に入る
```
docker-compose exec mySQLのコンテナ名 bash
```

※コンテナ名に関しては
```
docker ps
```
で確認可能。  
今回は`db`のはず…。動かない場合は確認してみてください。

## sqlへアクセスする。
```
mysql -u (user名) -p
```
この後パスワード入力する。

**docker内のDBにはアクセスできた！！**

## Connection refusedエラー
権限がないよと言われてる奴。  
docker内にuserに権限をつける用のファイルを設定。

```
echo "GRANT ALL ON *.* TO '"$DB_USER"'@'%' ;" | "${mysql[@]}"
echo 'FLUSH PRIVILEGES ;' | "${mysql[@]}"
```
![image](https://user-images.githubusercontent.com/84951254/145712354-fc85ecd6-0bc0-49a3-824e-428e5d9e4833.png)
https://obel.hatenablog.jp/entry/20201218/1608231600  

権限はもらえた。
```
docker network create network名
```
でネットワーク立ち上げてbuildする。
