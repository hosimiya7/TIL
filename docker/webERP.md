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

idコマンドの実行
```
$ id
uid=1000(hosimiya) gid=1000(hosimiya) groups=1000(hosimiya)…
```
いろいろファイル作成

.env(環境変数設定)
```
USERNAME=user
GROUPNAME=user

UID=1000
GID=1000
```

docker-compose(args以下追加)
```  
app:
    build:
      context: .
      dockerfile: ./docker/php/Dockerfile
      args:
          USERNAME: ${USERNAME}
          GROUPNAME: ${GROUPNAME}
          UID: ${UID}
          GID: ${GID}`

```

dockerfile(appのやつ)[参考](https://qiita.com/Spritaro/items/602118d946a4383bd2bb)
```
RUN groupadd -g $GID $GROUPNAME && \
    useradd -m -s /bin/bash -u $UID -g $GID $USERNAME
```

そしてエラー
```
 => ERROR [weberp_app stage-0 4/5] RUN apt-get update &&     curl -fsSL https://deb.nodesource.com/setup_16.x | bash - &&     apt-get -y install     27.8s 
------
 > [weberp_app stage-0 4/5] RUN apt-get update &&     curl -fsSL https://deb.nodesource.com/setup_16.x | bash - &&     apt-get -y install     nodejs    git     zip     unzip     vim     && docker-php-ext-install pdo_mysql bcmath     groupadd -g $GID $GROUPNAME &&     useradd -m -s /bin/bash -u $UID -g $GID $USERNAME:
#12 0.505 Get:1 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
#12 0.692 Get:2 http://security.debian.org/debian-security buster/updates/main amd64 Packages [309 kB]
#12 1.534 Get:3 http://deb.debian.org/debian buster InRelease [122 kB]
#12 1.578 Get:4 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
#12 3.023 Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7906 kB]
#12 3.272 Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [15.2 kB]
#12 4.035 Fetched 8470 kB in 4s (2299 kB/s)
#12 4.035 Reading package lists...
#12 4.779
#12 4.779 ## Installing the NodeSource Node.js 16.x repo...
#12 4.779
#12 4.781
#12 4.781 ## Populating apt-get cache...
#12 4.781
#12 4.781 + apt-get update
#12 4.862 Hit:1 http://security.debian.org/debian-security buster/updates InRelease
#12 4.865 Hit:2 http://deb.debian.org/debian buster InRelease
#12 4.895 Hit:3 http://deb.debian.org/debian buster-updates InRelease
#12 5.107 Reading package lists...
#12 5.610
#12 5.610 ## Installing packages required for setup: lsb-release gnupg...
#12 5.610
#12 5.611 + apt-get install -y lsb-release gnupg > /dev/null 2>&1
#12 11.92
#12 11.92 ## Confirming "buster" is supported...
#12 11.92
#12 11.92 + curl -sLf -o /dev/null 'https://deb.nodesource.com/node_16.x/dists/buster/Release'
#12 12.81
#12 12.81 ## Adding the NodeSource signing key to your keyring...
#12 12.81
#12 12.81 + curl -s https://deb.nodesource.com/gpgkey/nodesource.gpg.key | gpg --dearmor | tee /usr/share/keyrings/nodesource.gpg >/dev/null
#12 13.02
#12 13.02 ## Creating apt sources list file for the NodeSource Node.js 16.x repo...
#12 13.02
#12 13.02 + echo 'deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_16.x buster main' > /etc/apt/sources.list.d/nodesource.list
#12 13.02 + echo 'deb-src [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_16.x buster main' >> /etc/apt/sources.list.d/nodesource.list
#12 13.02
#12 13.02 ## Running `apt-get update` for you...
#12 13.02
#12 13.02 + apt-get update
#12 13.12 Hit:1 http://security.debian.org/debian-security buster/updates InRelease
#12 13.12 Hit:2 http://deb.debian.org/debian buster InRelease
#12 13.15 Hit:3 http://deb.debian.org/debian buster-updates InRelease
#12 13.17 Get:4 https://deb.nodesource.com/node_16.x buster InRelease [4584 B]
#12 13.72 Get:5 https://deb.nodesource.com/node_16.x buster/main amd64 Packages [768 B]
#12 13.73 Fetched 5352 B in 1s (7990 B/s)
#12 13.73 Reading package lists...
#12 14.24
#12 14.24 ## Run `sudo apt-get install -y nodejs` to install Node.js 16.x and npm
#12 14.24 ## You may also need development tools to build native addons:
#12 14.24      sudo apt-get install gcc g++ make
#12 14.24 ## To install the Yarn package manager, run:
#12 14.24      curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
#12 14.24      echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
#12 14.24      sudo apt-get update && sudo apt-get install yarn
#12 14.24
#12 14.24
#12 14.25 Reading package lists...
#12 14.77 Building dependency tree...
#12 14.90 Reading state information...
#12 15.08 The following additional packages will be installed:
#12 15.08   git-man less libcurl3-gnutls liberror-perl libgpm2 libpcre2-8-0 libx11-6
#12 15.08   libx11-data libxau6 libxcb1 libxdmcp6 libxext6 libxmuu1 openssh-client
#12 15.08   vim-common vim-runtime xauth xxd
#12 15.08 Suggested packages:
#12 15.08   gettext-base git-daemon-run | git-daemon-sysvinit git-doc git-el git-email
#12 15.08   git-gui gitk gitweb git-cvs git-mediawiki git-svn gpm keychain libpam-ssh
#12 15.08   monkeysphere ssh-askpass ctags vim-doc vim-scripts
#12 15.22 The following NEW packages will be installed:
#12 15.22   git git-man less libcurl3-gnutls liberror-perl libgpm2 libpcre2-8-0 libx11-6
#12 15.22   libx11-data libxau6 libxcb1 libxdmcp6 libxext6 libxmuu1 nodejs
#12 15.22   openssh-client unzip vim vim-common vim-runtime xauth xxd zip
#12 15.34 0 upgraded, 23 newly installed, 0 to remove and 44 not upgraded.
#12 15.34 Need to get 43.8 MB of archives.
#12 15.34 After this operation, 202 MB of additional disk space will be used.
#12 15.34 Get:1 http://deb.debian.org/debian buster/main amd64 less amd64 487-0.1+b1 [129 kB]
#12 15.37 Get:2 http://deb.debian.org/debian buster/main amd64 xxd amd64 2:8.1.0875-5 [140 kB]
#12 15.37 Get:3 https://deb.nodesource.com/node_16.x buster/main amd64 nodejs amd64 16.13.1-deb-1nodesource1 [25.8 MB]
#12 15.40 Get:4 http://deb.debian.org/debian buster/main amd64 vim-common all 2:8.1.0875-5 [195 kB]
#12 15.47 Get:5 http://deb.debian.org/debian buster/main amd64 openssh-client amd64 1:7.9p1-10+deb10u2 [782 kB]
#12 15.64 Get:6 http://deb.debian.org/debian buster/main amd64 libcurl3-gnutls amd64 7.64.0-4+deb10u2 [330 kB]
#12 15.75 Get:7 http://deb.debian.org/debian buster/main amd64 libpcre2-8-0 amd64 10.32-5 [213 kB]
#12 15.79 Get:8 http://deb.debian.org/debian buster/main amd64 liberror-perl all 0.17027-2 [30.9 kB]
#12 15.79 Get:9 http://deb.debian.org/debian buster/main amd64 git-man all 1:2.20.1-2+deb10u3 [1620 kB]
#12 16.22 Get:10 http://deb.debian.org/debian buster/main amd64 git amd64 1:2.20.1-2+deb10u3 [5633 kB]
#12 17.57 Get:11 http://deb.debian.org/debian buster/main amd64 libgpm2 amd64 1.20.7-5 [35.1 kB]
#12 17.58 Get:12 http://deb.debian.org/debian buster/main amd64 libxau6 amd64 1:1.0.8-1+b2 [19.9 kB]
#12 17.58 Get:13 http://deb.debian.org/debian buster/main amd64 libxdmcp6 amd64 1:1.1.2-3 [26.3 kB]
#12 17.58 Get:14 http://deb.debian.org/debian buster/main amd64 libxcb1 amd64 1.13.1-2 [137 kB]
#12 17.62 Get:15 http://deb.debian.org/debian buster/main amd64 libx11-data all 2:1.6.7-1+deb10u2 [299 kB]
#12 17.68 Get:16 http://deb.debian.org/debian buster/main amd64 libx11-6 amd64 2:1.6.7-1+deb10u2 [757 kB]
#12 17.88 Get:17 http://deb.debian.org/debian buster/main amd64 libxext6 amd64 2:1.3.3-1+b2 [52.5 kB]
#12 17.89 Get:18 http://deb.debian.org/debian buster/main amd64 libxmuu1 amd64 2:1.1.2-2+b3 [23.9 kB]
#12 17.89 Get:19 http://deb.debian.org/debian buster/main amd64 unzip amd64 6.0-23+deb10u2 [172 kB]
#12 17.95 Get:20 http://deb.debian.org/debian buster/main amd64 vim-runtime all 2:8.1.0875-5 [5775 kB]
#12 19.46 Get:21 http://deb.debian.org/debian buster/main amd64 vim amd64 2:8.1.0875-5 [1280 kB]
#12 19.82 Get:22 http://deb.debian.org/debian buster/main amd64 xauth amd64 1:1.0.10-1 [40.3 kB]
#12 19.83 Get:23 http://deb.debian.org/debian buster/main amd64 zip amd64 3.0-11+b1 [234 kB]
#12 21.64 debconf: delaying package configuration, since apt-utils is not installed
#12 21.66 Fetched 43.8 MB in 6s (6938 kB/s)
#12 21.68 Selecting previously unselected package less.
(Reading database ... 13683 files and directories currently installed.)
#12 21.69 Preparing to unpack .../00-less_487-0.1+b1_amd64.deb ...
#12 21.69 Unpacking less (487-0.1+b1) ...
#12 21.72 Selecting previously unselected package xxd.
#12 21.72 Preparing to unpack .../01-xxd_2%3a8.1.0875-5_amd64.deb ...
#12 21.73 Unpacking xxd (2:8.1.0875-5) ...
#12 21.77 Selecting previously unselected package vim-common.
#12 21.77 Preparing to unpack .../02-vim-common_2%3a8.1.0875-5_all.deb ...
#12 21.78 Unpacking vim-common (2:8.1.0875-5) ...
#12 21.83 Selecting previously unselected package openssh-client.
#12 21.83 Preparing to unpack .../03-openssh-client_1%3a7.9p1-10+deb10u2_amd64.deb ...
#12 21.84 Unpacking openssh-client (1:7.9p1-10+deb10u2) ...
#12 21.92 Selecting previously unselected package libcurl3-gnutls:amd64.
#12 21.93 Preparing to unpack .../04-libcurl3-gnutls_7.64.0-4+deb10u2_amd64.deb ...
#12 21.93 Unpacking libcurl3-gnutls:amd64 (7.64.0-4+deb10u2) ...
#12 21.98 Selecting previously unselected package libpcre2-8-0:amd64.
#12 21.98 Preparing to unpack .../05-libpcre2-8-0_10.32-5_amd64.deb ...
#12 21.99 Unpacking libpcre2-8-0:amd64 (10.32-5) ...
#12 22.03 Selecting previously unselected package liberror-perl.
#12 22.03 Preparing to unpack .../06-liberror-perl_0.17027-2_all.deb ...
#12 22.04 Unpacking liberror-perl (0.17027-2) ...
#12 22.06 Selecting previously unselected package git-man.
#12 22.06 Preparing to unpack .../07-git-man_1%3a2.20.1-2+deb10u3_all.deb ...
#12 22.07 Unpacking git-man (1:2.20.1-2+deb10u3) ...
#12 22.17 Selecting previously unselected package git.
#12 22.17 Preparing to unpack .../08-git_1%3a2.20.1-2+deb10u3_amd64.deb ...
#12 22.19 Unpacking git (1:2.20.1-2+deb10u3) ...
#12 22.66 Selecting previously unselected package libgpm2:amd64.
#12 22.66 Preparing to unpack .../09-libgpm2_1.20.7-5_amd64.deb ...
#12 22.67 Unpacking libgpm2:amd64 (1.20.7-5) ...
#12 22.70 Selecting previously unselected package libxau6:amd64.
#12 22.70 Preparing to unpack .../10-libxau6_1%3a1.0.8-1+b2_amd64.deb ...
#12 22.70 Unpacking libxau6:amd64 (1:1.0.8-1+b2) ...
#12 22.74 Selecting previously unselected package libxdmcp6:amd64.
#12 22.74 Preparing to unpack .../11-libxdmcp6_1%3a1.1.2-3_amd64.deb ...
#12 22.74 Unpacking libxdmcp6:amd64 (1:1.1.2-3) ...
#12 22.77 Selecting previously unselected package libxcb1:amd64.
#12 22.78 Preparing to unpack .../12-libxcb1_1.13.1-2_amd64.deb ...
#12 22.78 Unpacking libxcb1:amd64 (1.13.1-2) ...
#12 22.81 Selecting previously unselected package libx11-data.
#12 22.81 Preparing to unpack .../13-libx11-data_2%3a1.6.7-1+deb10u2_all.deb ...
#12 22.81 Unpacking libx11-data (2:1.6.7-1+deb10u2) ...
#12 22.88 Selecting previously unselected package libx11-6:amd64.
#12 22.88 Preparing to unpack .../14-libx11-6_2%3a1.6.7-1+deb10u2_amd64.deb ...
#12 22.88 Unpacking libx11-6:amd64 (2:1.6.7-1+deb10u2) ...
#12 22.96 Selecting previously unselected package libxext6:amd64.
#12 22.96 Preparing to unpack .../15-libxext6_2%3a1.3.3-1+b2_amd64.deb ...
#12 22.96 Unpacking libxext6:amd64 (2:1.3.3-1+b2) ...
#12 23.00 Selecting previously unselected package libxmuu1:amd64.
#12 23.00 Preparing to unpack .../16-libxmuu1_2%3a1.1.2-2+b3_amd64.deb ...
#12 23.00 Unpacking libxmuu1:amd64 (2:1.1.2-2+b3) ...
#12 23.02 Selecting previously unselected package nodejs.
#12 23.03 Preparing to unpack .../17-nodejs_16.13.1-deb-1nodesource1_amd64.deb ...
#12 23.03 Unpacking nodejs (16.13.1-deb-1nodesource1) ...
#12 25.20 Selecting previously unselected package unzip.
#12 25.20 Preparing to unpack .../18-unzip_6.0-23+deb10u2_amd64.deb ...
#12 25.20 Unpacking unzip (6.0-23+deb10u2) ...
#12 25.25 Selecting previously unselected package vim-runtime.
#12 25.25 Preparing to unpack .../19-vim-runtime_2%3a8.1.0875-5_all.deb ...
#12 25.26 Adding 'diversion of /usr/share/vim/vim81/doc/help.txt to /usr/share/vim/vim81/doc/help.txt.vim-tiny by vim-runtime'
#12 25.27 Adding 'diversion of /usr/share/vim/vim81/doc/tags to /usr/share/vim/vim81/doc/tags.vim-tiny by vim-runtime'
#12 25.27 Unpacking vim-runtime (2:8.1.0875-5) ...
#12 25.79 Selecting previously unselected package vim.
#12 25.79 Preparing to unpack .../20-vim_2%3a8.1.0875-5_amd64.deb ...
#12 25.80 Unpacking vim (2:8.1.0875-5) ...
#12 25.91 Selecting previously unselected package xauth.
#12 25.91 Preparing to unpack .../21-xauth_1%3a1.0.10-1_amd64.deb ...
#12 25.92 Unpacking xauth (1:1.0.10-1) ...
#12 25.94 Selecting previously unselected package zip.
#12 25.95 Preparing to unpack .../22-zip_3.0-11+b1_amd64.deb ...
#12 25.95 Unpacking zip (3.0-11+b1) ...
#12 26.00 Setting up libxau6:amd64 (1:1.0.8-1+b2) ...
#12 26.01 Setting up libxdmcp6:amd64 (1:1.1.2-3) ...
#12 26.02 Setting up libxcb1:amd64 (1.13.1-2) ...
#12 26.03 Setting up libgpm2:amd64 (1.20.7-5) ...
#12 26.04 Setting up openssh-client (1:7.9p1-10+deb10u2) ...
#12 26.11 Setting up unzip (6.0-23+deb10u2) ...
#12 26.13 Setting up less (487-0.1+b1) ...
#12 26.20 debconf: unable to initialize frontend: Dialog
#12 26.20 debconf: (TERM is not set, so the dialog frontend is not usable.)
#12 26.20 debconf: falling back to frontend: Readline
#12 26.22 Setting up libcurl3-gnutls:amd64 (7.64.0-4+deb10u2) ...
#12 26.23 Setting up nodejs (16.13.1-deb-1nodesource1) ...
#12 26.24 Setting up xxd (2:8.1.0875-5) ...
#12 26.25 Setting up liberror-perl (0.17027-2) ...
#12 26.26 Setting up zip (3.0-11+b1) ...
#12 26.27 Setting up vim-common (2:8.1.0875-5) ...
#12 26.29 Setting up libx11-data (2:1.6.7-1+deb10u2) ...
#12 26.30 Setting up libpcre2-8-0:amd64 (10.32-5) ...
#12 26.31 Setting up git-man (1:2.20.1-2+deb10u3) ...
#12 26.31 Setting up libx11-6:amd64 (2:1.6.7-1+deb10u2) ...
#12 26.32 Setting up vim-runtime (2:8.1.0875-5) ...
#12 26.39 Setting up libxmuu1:amd64 (2:1.1.2-2+b3) ...
#12 26.40 Setting up vim (2:8.1.0875-5) ...
#12 26.41 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vim (vim) in auto mode
#12 26.41 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vimdiff (vimdiff) in auto mode
#12 26.42 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rvim (rvim) in auto mode
#12 26.42 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/rview (rview) in auto mode
#12 26.42 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/vi (vi) in auto mode
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/da/man1/vi.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/de/man1/vi.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/fr/man1/vi.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/it/man1/vi.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/ja/man1/vi.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/pl/man1/vi.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/ru/man1/vi.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: warning: skip creation of /usr/share/man/man1/vi.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group vi) doesn't exist
#12 26.42 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/view (view) in auto mode
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/da/man1/view.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/de/man1/view.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/fr/man1/view.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/it/man1/view.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ja/man1/view.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/pl/man1/view.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ru/man1/view.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link 
group view) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/man1/view.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group 
view) doesn't exist
#12 26.43 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/ex (ex) in auto mode
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/da/man1/ex.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/de/man1/ex.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/fr/man1/ex.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/it/man1/ex.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ja/man1/ex.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/pl/man1/ex.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ru/man1/ex.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/man1/ex.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group ex) doesn't exist
#12 26.43 update-alternatives: using /usr/bin/vim.basic to provide /usr/bin/editor (editor) in auto mode
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/da/man1/editor.1.gz because associated file /usr/share/man/da/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/de/man1/editor.1.gz because associated file /usr/share/man/de/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/fr/man1/editor.1.gz because associated file /usr/share/man/fr/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/it/man1/editor.1.gz because associated file /usr/share/man/it/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ja/man1/editor.1.gz because associated file /usr/share/man/ja/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/pl/man1/editor.1.gz because associated file /usr/share/man/pl/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/ru/man1/editor.1.gz because associated file /usr/share/man/ru/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.43 update-alternatives: warning: skip creation of /usr/share/man/man1/editor.1.gz because associated file /usr/share/man/man1/vim.1.gz (of link group editor) doesn't exist
#12 26.44 Setting up libxext6:amd64 (2:1.3.3-1+b2) ...
#12 26.46 Setting up git (1:2.20.1-2+deb10u3) ...
#12 26.49 Setting up xauth (1:1.0.10-1) ...
#12 26.50 Processing triggers for libc-bin (2.28-10) ...
#12 26.51 Processing triggers for mime-support (3.62) ...
#12 27.55 getopt: invalid option -- 'g'
#12 27.55 usage: /usr/local/bin/docker-php-ext-install [-jN] ext-name [ext-name ...]
#12 27.55    ie: /usr/local/bin/docker-php-ext-install gd mysqli
#12 27.55        /usr/local/bin/docker-php-ext-install pdo pdo_mysql
#12 27.55        /usr/local/bin/docker-php-ext-install -j5 gd mbstring mysqli pdo pdo_mysql shmop
#12 27.55
#12 27.55 if custom ./configure arguments are necessary, see docker-php-ext-configure
#12 27.55
#12 27.55 Possible values for ext-name:
#12 27.58 bcmath bz2 calendar ctype curl dba dom enchant exif ffi fileinfo filter ftp gd gettext gmp hash iconv imap intl json ldap mbstring mysqli oci8 odbc opcache pcntl pdo pdo_dblib pdo_firebird pdo_mysql pdo_oci pdo_odbc pdo_pgsql pdo_sqlite pgsql phar posix pspell readline reflection session shmop simplexml snmp soap sockets sodium spl standard sysvmsg sysvsem sysvshm tidy tokenizer xml xmlreader xmlrpc xmlwriter xsl zend_test zip
#12 27.58
#12 27.58 Some of the above modules are already compiled into PHP; please check
#12 27.58 the output of "php -i" to see which modules are already loaded.
------

failed to solve: rpc error: code = Unknown desc = executor failed running [/bin/sh -c apt-get update &&     curl -fsSL https://deb.nodesource.com/setup_16.x | bash - &&     apt-get -y install     nodejs    git     zip     unzip     vim     && docker-php-ext-install pdo_mysql bcmath     groupadd -g $GID $GROUPNAME &&     useradd -m -s /bin/bash -u $UID -g $GID $USERNAME]: exit code: 1
```
rpcエラー…?
リモートプロシージャコールのエラー…というものらしい。
