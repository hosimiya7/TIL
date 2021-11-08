# dockerの構築(wordpress)
## dockerのインストール

## dockerHub

## wordpress
https://53ningen.com/docker-wordpress/

### dockerfile/docker-compose

### サーバー

・アプリケーションサーバー
・webサーバー
・DBサーバー

それぞれにフォルダを作って、その中にDockerfileを作成

（ubuntuにすべてのサーバーをぶち込むこともできる。今回はやらない。その場合はdockerfileは1つ）

これを繋ぐのがdocker-compose

FROM (名前):(タグ…バージョン)
※バージョンなしだと最新→ローカルと本番で変わる可能性あるから指定したほうがいい
FROM mysql:5.7
FROM nginx
FROM wordpress
