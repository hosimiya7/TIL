# 表示までの流れ
### app.jsにVueファイルを書く  
```Vue.component('top', require('./components/top.vue').default);```  
### viewファイル（blade）にcsrf_token
```<meta name="csrf-token" content="{{ csrf_token() }}">```  
### viewファイルのid=”app”が必要
1階層目のdivにつける
### jsファイルの読み込み
```<script src="{{ asset('js/app.js') }}" defer></script>```
