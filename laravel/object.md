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

### selfとthis

selfはクラス自身  
thisはインスタンス化したもの自身

model(だめなやつ)
```
// Error
// Using $this when not in object contextがでる
// インスタンス化してないので$thisは呼び出せない(？)

class Club extends Model
{
    use HasFactory;
    // 定数定義
    const INSUFFICIENT = 0;
    const UNAPPROVED = 1;
    const APPROVED = 2;
    
    public function isInsufficient()
    {
        // self::INSUFFICIENT 定数を自身から呼び出している
        // $thisは「このインスタンスの」の意味だけど、Clubclassはbladeでインスタンス化してないのでデータがない
        self::INSUFFICIENT === $this->approval;
    }

}

```

blade
```
<li>{{ $club->name }}
    <span>
        {{-- @if ($club->approval === App\Models\Club::INSUFFICIENT) --}}
        @if (App\Models\Club::isInsufficient())
            人数不足
        @elseif($club->approval === App\Models\Club::UNAPPROVED)
            未承認
            <form method="POST" action="{{ route('club.approval') }}">
                @csrf
                <input type="hidden" name="club_id" value="{{ $club->id }}">
                <input type="submit" value="承認する">
            </form>
        @elseif($club->approval === App\Models\Club::APPROVED)
            承認済み
        @endif
    </span>
</li>
```

~~bladeでインスタンス化する…？~~  
Club::allで持ってきたデータはcollectionオブジェクトになるので、controller経由だとオブジェクトが持ってこれる。
なんかserviceってやつもあるらしい…(よくわかってない)

model(正しいやつ)

```
class Club extends Model
{
    use HasFactory;

    const INSUFFICIENT = 0;
    const UNAPPROVED = 1;
    const APPROVED = 2;
    
    public function isInsufficient()
    {
        return $this->approval === self::INSUFFICIENT;
    }
}
```
blade
```
<li>{{ $club->name }}
    <span>
        {{-- @if ($club->approval === App\Models\Club::INSUFFICIENT) --}}
    <!-- clubはコントローラー経由で持ってきたオブジェクト -->
        @if ($club->isInsufficient())
            人数不足
        @elseif($club->approval === App\Models\Club::UNAPPROVED)
            未承認
            <form method="POST" action="{{ route('club.approval') }}">
                @csrf
                <input type="hidden" name="club_id" value="{{ $club->id }}">
                <input type="submit" value="承認する">
            </form>
        @elseif($club->approval === App\Models\Club::APPROVED)
            承認済み
        @endif
    </span>
</li>
```

controller
```
class StudentController extends Controller
{
    public function show()
    {
        $students = Student::all();
        // $clubはcollectionオブジェクトになる
        $clubs = Club::all();
        return view('welcome', ['students' => $students, 'clubs' => $clubs]);
    }
}

```

## laravelのクラス・オブジェクト
laravelに既に存在しているクラスたち

### controllerクラス
controllerの継承元

### builderクラス
実行前のクラス。いったい何が含まれているのか…？？
```
// getしてない状態
$club = Club::find($request->club_id);
```

### Eloquentクラス
modelの継承元  
関数にDBからデータを取得したり、DBにデータを渡したりするものがある。
* save()
* get()
* insert()

### collectionオブジェクト
DBからデータを取得した時点でcollectionオブジェクトになる。  
何かのクラスからインスタンス化されたデータの入ったオブジェクト…。  
何かってなんだ！  
```
Member::where('club_id', $club->id)->get()
```
