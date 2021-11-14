
# 親子間でのデータの受け渡しに関しての気づき

## 最初に

### やらかし内容
もちべったーを作る上で、様々なコンポーネント間で共有したいDBからの値をstoreに入れて管理していた。

### 問題点
どこでも変更ができるグローバルな変数になってしまうため、エラーが出た際にどこで不具合が出たかわかりにくい。  
よってstoreで数値が頻繁に変わる変数(動的なデータ？)を管理しない方がいい。

### 改善策
storeに入れて一括管理していた変数を親から子へのデータの受け渡しをするように変更する。  

## やってみたこと

### 親から子へのデータ受け渡し
v-bindとpropsを使う。 

#### 親

親コンポーネントで送りたい値を、子コンポーネントに伝える。 
 <子コンポーネント名 :属性名="変数名" />   

ex  
```
// 親
<template>
  <div id="app">
    <h2>親子間</h2>
    <Child :msg="toChildMsg" />
  </div>
</template>

<script>
import Child from './components/Child'

export default {
  name: 'App',
  components: {
    Child,
  },
  data() {
    return {
      toChildMsg: "parent message!",
    }
  },
}
</script>

```
#### 子
親から送られたデータをpropsで受け取る  
props:[親で指定した属性名]  
```
// 子
<template>
  <div>
    <h3>Child</h3>
    <p>{{ msg }}</p>
  </div>
</template>

<script>
export default {
  name: "Child",
  props: ["msg"],
};
</script>

```

### 子から親へのデータ渡し
#### 子
親へ $emit を使ってデータを渡す。
```
<template>
    <div>
        <button @click="$emit('panretMessage')">message</button>
    </div>
</template>

```

#### 親
子から送られたデータをv-onで受け取る。
```
<template>
    <div>
        <child v-on:panretMessage="messageLog"></child>
    </div>
</template>
<script>
import Child from './child.vue';
export default {
    components: {
        Child
    },
    methods: {
        messageLog() {
            console.log("parent");
        }
    }
}
</script>
```

## そしてエラー…
```Avoid mutating a prop directly```
こんなエラーを吐きました。内容としては親から送られてきたデータは子では編集できないよっていう感じ。
子の側でもデータの更新をする場合があるので親子両方でデータの更新ができる状態にしたかった。
というわけで再び検索…。

### syncを使う
syncは親から受け取ったデータを、propsで定義した変数のみで変更する方法。

親から子へ送る際に`<子コンポーネント名 :属性名.sync="変数名" />`にする

#### 親
```
<SyncChiled :message.sync="msg" />
```

### 参照先
[Vue.js における Component 間のデータの受け渡しまとめ](https://qiita.com/att55/items/91b683c68b5057eaac51)
[Vue.jsでコンポーネント親子間の値の受け渡し](https://qiita.com/y_sasaki/items/5bbed5439fcfef9f8c40)
[【Vue】Avoid mutating a prop directlyエラーの発生原因と対処法](https://qiita.com/shizen-shin/items/ec05071140b9d5d7d31a)
