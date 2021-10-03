# laravel新規プロジェクト作成

### プロジェクト作成  
```composer create-project laravel/laravel プロジェクト名 --prefer-dist "6.*"```  
この場合 version 6 の最新が入る(最後の部分でバージョン管理)  

### 起動  
```php artisan serve```　

### vueを入れるなら…
```$ composer require laravel/ui 1.*```  
バージョン指定しないとうまくいかない。  
```npm install```  
```npm run dev```  
```php artisan ui vue```  
再度```npm install```

SPAを作るなら
```npm install --save vue-router```

### jsのパッケージをpackage.jsonに記入
ex:  
   "jquery": "^3.2",  

### jsを出力
```npm install```  
package.jsonに書かれたものがインストールされる  

### jsコンパイル
```npm run dev```  
走らせるとコンパイルされる  
```npm run watch```  
変更があると自動でコンパイルされる
