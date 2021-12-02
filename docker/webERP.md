# webERPのdockerのwindows動作確認
[webERP](https://github.com/CoopTechOrg/WebERP)のdocker動作確認した

## Build & Up
docker-compose up -d --build

### エラー発生
案の定出ました
```
ERROR: for db  Cannot start service db: Ports are not available: listen tcp 0.0.0.0:3306: bind: Only one usage of each socket address (protocol/network address/port) is normally permitted.
```
訳すと「サービスが始まらないよ。3306ポートが使えない。各ソケットアドレス（プロトコル/ネットワークアドレス/ポート）は、一つだけしか使えない」  
3306ポートが何かと被ってるっぽい。

### 解消したい…
まずdockerを止める。
```
docker-compose down
```

3306が何に使われてるのか確認する
```
netstat -ano | findstr ":3306"
```

#### コード内容 
##### netstat
アクティブな TCP 接続をコンピューターがリッスンしている、  
イーサネットの統計情報、IP ルーティング テーブル、IPv4 プロトコルの統計情報 (、IP、ICMP、TCP、および UDP)、  
および IPv6 の統計情報 (の IPv6、ICMPv6、IPv6 経由で TCP および UDP IPv6 プロトコル経由で) ポートを表示します。   
パラメーターなしで使用され、このコマンドはアクティブな TCP 接続を表示します。  

##### netstat -a
すべてのアクティブな TCP 接続と、コンピューターがリッスンする TCP および UDP ポートが表示されます。

##### netstat -n
アクティブな TCP 接続の表示、ただし名前を決定する試行が行われません。

##### netstat -o
アクティブな TCP 接続が表示され、接続ごとにプロセス ID (PID) が含まれています。   
Windows タスク マネージャーで [プロセス] タブには、PID に基づくアプリケーションが表示されます。  
このパラメーターと組み合わせることができます -a, 、-n, 、および -pします。  

##### netstat -an | findstr ":ポート番号 "
ポート番号を指定するコマンド。

返答
```
  TCP         0.0.0.0:3306           0.0.0.0:0              LISTENING       6688
  TCP         0.0.0.0:33060          0.0.0.0:0              LISTENING       6688
  TCP         [::]:3306              [::]:0                 LISTENING       6688
  TCP         [::]:33060             [::]:0                 LISTENING       6688
```
6688のプロセスが怪しい
```
tasklist /FI "PID eq 6688"
```
tasklistはwindows系のコマンド  
コマンドプロンプトで実行

返答
```
イメージ名                     PID セッション名     セッション# メモリ使用量
========================= ======== ================ =========== ============
mysqld.exe                    6688 Services                   0     44,772 K
```

mysqld.exeが悪さしてるぽいけど、これなんだ…？  
[mysqld.exe](https://www.processlibrary.com/ja/directory/files/mysqld/402544/)  
MySQLを使ってたらデータが溜まる的な感じかな？  
システムプロセスではないプロセス…。  

~~タスクマネージャーからタスクを止めます。~~(これはダメでした。)  
.envのポート番号を変更します。  

いけた…！


## Cドライブ下でやってました。
通りで重たいわけです。ダメでした。  
wslで動かします　[使い方](https://github.com/hosimiya7/TIL/blob/main/docker/howToUse.md)  

### 第一のエラー
```
WARNING: The DB_PORT variable is not set. Defaulting to a blank string.
WARNING: The DB_NAME variable is not set. Defaulting to a blank string.
WARNING: The DB_USER variable is not set. Defaulting to a blank string.
WARNING: The DB_PASSWORD variable is not set. Defaulting to a blank string.
WARNING: The WEB_PORT variable is not set. Defaulting to a blank string.
WARNING: The DB_ROOT_PASSWORD variable is not set. Defaulting to a blank string.
ERROR: The Compose file '.\docker-compose.yml' is invalid because:
services.db.ports contains an invalid type, it should be a number, or an object
services.web.ports contains an invalid type, it should be a number, or an object
```

いろんなものがセットされてないよ～とのこと。

.envを作っていなかった。  
.env.exampleから複製し、ポート番号を8001と3300に変更。

### 第2のエラー
```
unable to prepare context: path "\\\\?\\\\\\wsl$\\Ubuntu-20.04\\home\\hosimiya\\dev\\webERP" not found
ERROR: Service 'app' failed to build : Build failed
```

パスがないよ～、だからappが立ち上げられないよ～って言われてる。
パスをどうにかしないと…  

ネットワークディレクトリ(\\)はwindouwにしかないし、ターミナル(？)でも対応してないものが多い  
つまりgit bashだと表示できない

wslでwindowsの中に入ることで実行できる。

### docker立ち上がりました！！

## appコンテナの中に入る

```
docker-compose exec app bash
```
入れた。

いろいろ設定する。[webERP](https://github.com/CoopTechOrg/WebERP)  
Cドライブでやって時より断然軽い！！最強！！

### 第3のエラー
```
SQLSTATE[HY000] [2002] Connection refused
(SQL: select * from information_schema.tables where table_schema = db_name and table_name = migrations and table_type = 'BASE TABLE')
```

```
php artisan migrate
```
で出たエラー。
laravelの.envとdockerの.envがあってないぽい？  
dockerのポート番号を3300に変えたしなぁ  

[これっぽい](https://qiita.com/isaatsu0131/items/50f8dca389b60a1fd5b5)

laravelの.envをいじろうとすると…

### 第4のエラー
十分な権限がありません。と言われました…。

![image](https://user-images.githubusercontent.com/84951254/144355953-a1f81306-aa13-4ff5-b5b4-989261f9a8c2.png)  
管理者権限  
![image](https://user-images.githubusercontent.com/84951254/144355909-d900834c-8251-4ec9-bbad-6f1b6df2b463.png)  


[参考:Docker for WindowsとWSLを併用するときのパーミッションとファイルユーザ](https://zenn.dev/rotelstift/articles/docker-for-windows-permission)  
[参考:【Docker】 WSL 2 を利用したコンテナー内開発で権限をどう設定するべきか](https://futureys.tokyo/how-permission-should-be-set-for-developing-inside-a-container-using-wsl-2)  

dockerの想定しているuser権限(root)とwslのuser権限(hosimiya)が違うために権限がないといわれていそう。

wsl  
![image](https://user-images.githubusercontent.com/84951254/144361467-96ac52a6-02a9-409b-aa57-6b840e3d6f6a.png)  
dockerのコンテナ内  
![image](https://user-images.githubusercontent.com/84951254/144361923-056caeef-c879-4ca7-800b-37c6e9b4a7ab.png)

やはりな…

#### user権限を一致させたい

* dockerをwslに寄せる  
* wslをdockerに寄せる  
* 一般userでも使えるようにする…？  

https://qiita.com/Spritaro/items/602118d946a4383bd2bb
