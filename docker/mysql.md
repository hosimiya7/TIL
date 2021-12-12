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
docker exec -it mySQLのコンテナ名　bash
```

※コンテナ名に関しては
```
docker ps
```
で確認可能。  
今回は`db`のはず…。動かない場合は確認してみてください。
