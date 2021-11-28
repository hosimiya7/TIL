# オブジェクト指向について
ソフトウェア開発とコンピュータプログラミングのために用いられる考え方。
まだあやふやだけど自分のわかっていることをまとめる。

## 用語
### クラス  
設計図に例えられる。
たくさんの変数や関数が書かれている。

### インスタンス化 
クラスを使える状態にしたもの。
これにより、複製が可能になる。
クラス内で定義した変数にあとから値を代入できるようになる。

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
// 使うクラスは記載
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
        // insertはEloquentクラスが元から持っている関数。それがClubクラスに継承されてここで使われている。
        Club::insert(['name' => $name]);
        return redirect('/');
    }

}

```

```
// これは後々…。
class Club extends Model
{
    use HasFactory;
    // 定数定義
    const INSUFFICIENT = 0;
    const UNAPPROVED = 1;
    const APPROVED = 2;

    public function students()
    {
        return $this->belongsToMany(Student::class, 'members');
    }

    public function isInsufficient()
    {
        // tureだったら
        self::INSUFFICIENT === $this->approval;
    }

    public function isApproved()
    {
    }
}

```

~~インスタンス化せずにクラスから直接変数関数が呼び出せるが、インスタンス化した場合との違いが良くわからない。  
クラスメソッド・クラスプロパティのメリット・デメリットは？  
ここでいう静的って何だろう。値が確実に変わらないこと？~~

インスタンス化することで使いまわせるようになる。  
クラス内で決められた定数や関数しか使わない場合は静的関数・静的変数でよい。  
新しく値を代入したり、その代入した値を使っている関数はインスタンス化しないと使えない。  
使いまわしできるから便利よねって話。

### クラスメソッド・クラスプロパティ
クラスから直接値や関数を呼び出す＝変更不可
呼び出すときは クラス名::メソッド名
```
Club::insert(['name' => $name]);
```

### インスタンスメソッド
インスタンス化したものから呼び出す＝再代入可
呼び出すときは $変数 = new クラス名(); でインスタンス化。  
$変数->メソッド名
```
$mainCursor = new MainCursor();
// mainCursorのインスタンスに書かれた関数isKeyDownを呼び出して実行する
$mainCursor->isKeyDown()
```
