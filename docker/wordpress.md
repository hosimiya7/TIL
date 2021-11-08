# dockerの構築(wordpress)
## dockerのインストール
https://kahoo.blog/howto-wsl2-docker-install/
## dockerhub
https://hub.docker.com/
## wordpress環境構築
https://53ningen.com/docker-wordpress/

## dockerfile/docker-composeの作成(ex.wordpress環境構築)

### サーバー

* アプリケーションサーバー(wordpress)  
* webサーバー(nginx)  
* DBサーバー(mysql)  

それぞれにフォルダを作って、その中にDockerfileを作成

（ubuntuにすべてのサーバーをぶち込むこともできる。今回はやらない。その場合はdockerfileは1つ）

これを繋ぐのが`docker-compose`

### dockerfile
FROM (名前):(タグ…バージョン)  
※バージョンなしだと最新→ローカルと本番で変わる可能性あるから指定したほうがいい  
`FROM mysql:5.7`  
`FROM nginx`  
`FROM wordpress`  

### docker-compose
```
version: '3'

services:

  mysql:
    build: ./mysql
    ports:
      - '${MYSQL_PORT}:3306'
    volumes:
      - ./mysql/volumes:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_USER=${MYSQL_USER}
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
    restart: always

  nginx:
    build: ./nginx
    ports:
      - '${NGINX_HOST_PORT_80}:80'
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./nginx/log:/var/log/nginx
      - ./wordpress/src:/var/www/html
    depends_on:
      - wordpress
    restart: always

  wordpress:
    build: ./wordpress
    volumes:
      - ./wordpress/src:/var/www/html
    environment:
      - WORDPRESS_DB_HOST=${WORDPRESS_DB_HOST}
      - WORDPRESS_DB_USER=${WORDPRESS_DB_USER}
      - WORDPRESS_DB_PASSWORD=${WORDPRESS_DB_PASSWORD}
      - WORDPRESS_DB_NAME=${WORDPRESS_DB_NAME}
    depends_on:
      - mysql
    restart: always

```

version: dockerのバージョン  
build: どこのdockerfileを参照する  
volumes: localの場所:リモートの場所  
environment: 環境変数の設定(.envに記載)  
depends_on: 記載されたものの後に立ち上がる  
restart: コンテナを自動的に起動する

## コマンド
docker-compose  
ps プロセス(現在起動しているものを確認)  
build (構築)  
up (走らせる)  
up --build(構築して走らせる)  
down(止める)  

https://qiita.com/tanakin_prog/items/6e6219a62e7a05eb22c2

## その他
.envを作成。(環境変数)  
nginx.confを作成。
