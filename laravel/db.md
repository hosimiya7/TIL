## table作成
***php artisan make:migration create_users_table***

### カラム追加時
(post_tableにuser_idを追加)  
***php artisan make:migration add_user_id_to_posts_table --table=posts***  


## 実行
***php artisan migrate***


## ロールバック
***php artisan migrate:rollback***


## mastertable
seederでデータを入力しておく。  
***php artisan make:seeder UsersTableSeeder***

？　大量のデータを保存するには、効率のいい書き方が必要だけど…
