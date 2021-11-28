# オブジェクト指向について
ソフトウェア開発とコンピュータプログラミングのために用いられる考え方。
まだあやふやだけど自分のわかっていることをまとめる。

## 用語
### クラス  
設計図に例えられる。
たくさんの変数や関数が書かれている。

### インスタンス化 
クラスを使える状態にしたもの。
これにより、クラス内に書かれている変数や関数を使えるようになる。

```
// インスタンス化したものが、this.mainCursor(オブジェクト変数)の中に入っている。
$this->mainCursor = new MainCursor();
// mainCursorのインスタンスに書かれた定数INDENT1を呼び出す。
$this->mainCursor->INDENT1
```

### オブジェクト
「物」「対象」  
変数(プロパティ)と関数(メソッド)をもったもの。  
クラスをインスタンス化したことにより、実際に使えるようになった変数と関数の集合体(？)。  
ここから変数と関数を呼び出して使う。  
呼び出し方はインスタンス化の項目参照。  
 
### プロパティ  
クラスの中に書かれた変数

### メソッド  
クラスの中に書かれた関数

## laravelのオブジェクト指向
### controller・model
それぞれクラス。つまり設計書の状態。  
派生するcontrollerは、元のcontrollerを継承している。  
modelも同様。  

```
// 使うクラスは記載→　ここに書くことで継承されている？
use App\Models\Club;
use Illuminate\Http\Request;

// Controllerというクラスを継承したClubControllerというクラス
class ClubController extends Controller
{
    //createという関数(メソッド)
    public function create(Request $request)
    {
        $name = $request->club;
        // modelのClubというクラスから、インスタンス化せずに直接関数を持ってきている。
        // insertはmodelクラスが元から持っている関数。それがClubクラスに継承されてここで使われている。(多分)
        Club::insert(['name' => $name]);
        return redirect('/');
    }

}

```

**インスタンス化せずにクラスから直接変数関数が呼び出せるが、インスタンス化した場合との違いが良くわからない。  
クラスメソッド・クラスプロパティのメリット・デメリットは？  
ここでいう静的って何だろう。値が確実に変わらないこと？**  

### クラスメソッド・クラスプロパティ
後々記述予定
呼び出すときは クラス名::メソッド名
```
Club::insert(['name' => $name]);
```

### インスタンスメソッド
インスタンス化したものから呼び出す
呼び出すときは $変数 = new クラス名(); でインスタンス化。  
$変数->メソッド名
```
$mainCursor = new MainCursor();
// mainCursorのインスタンスに書かれた関数isKeyDownを呼び出して実行する
$mainCursor->isKeyDown()
```
