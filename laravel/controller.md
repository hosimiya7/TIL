# controllerを文章を短く(fill)
fillを使うことで、カラムと同じ名前のデータを自動で振り分けてくれる。  
名前が違うと使えない。

### 元のコード

        // $user = Auth::user();
        // $user->goal->goal = $request->goal;
        // $user->goal->number = $request->number;
        // $user->goal->unit = $request->unit;
        // $user->goal->user_id = Auth::id();
        // $user->goal->save();
        
### 短くする

```
$user = Auth::user();
$user->goal->fill($request->all())->save();
```
 
