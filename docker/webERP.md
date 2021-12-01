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
