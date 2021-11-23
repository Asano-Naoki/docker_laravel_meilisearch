# docker_laravel_meilisearch
meilisearchを使えるLaravel環境を作るためのdocker-compose

## 概要
docker-composeを使って最小限のmeilisearch付きLaravel(LEMP)環境を構築することができます。

## 前提条件
(まず https://github.com/Asano-Naoki/docker_laravel_minimum を試すことをおすすめします。)

このリポジトリを使う前に、モデル（データベース）機能のあるLaravelプロジェクトディレクトリを用意してください。

新しいLaravelプロジェクトディレクトリを作る場合は、以下の手順に従ってください。

1. まっさらなLaravelプロジェクトディレクトリを作ります。
```
docker run --rm -v $PWD:/app composer create-project laravel/laravel example-meilisearch-app
```
2. .envファイルを編集します。
```
-DB_HOST=127.0.0.1
+DB_HOST=mysql
-DB_USERNAME=root
-DB_PASSWORD=
+DB_USERNAME=test_user
+DB_PASSWORD=test_pass
+SCOUT_DRIVER=meilisearch
+SCOUT_QUEUE=true
+MEILISEARCH_HOST=http://meilisearch:7700
+MEILISEARCH_KEY=masterKey
```
3.日本語のダミーデータを作成するために、config/app.phpのfaker_localeの値をja_JPに変更します。
```
-'faker_locale' => 'en_US',
+'faker_locale' => 'ja_JP',
```

3. このリポジトリ内のファイルとディレクトリをすべて、gitコマンド（fetchとmerge）か手動で、Laravelプロジェクトディレクトリにコピーします。
```
git init
git add .
git commit -m "first commit"
git remote add origin git@github.com:Asano-Naoki/docker_laravel_meilisearch.git
git fetch
git merge origin/main --allow-unrelated-histories
```
4. docker-composeを開始します。
```
docker-compose up -d
```
5. usersテーブルを作ります。
```
docker-compose exec php sh
...
(after entering sh of php)
...
php artisan migrate
```
注: 最初のセットアップ時には、mysqlファイルがすべて作られていないために、php artisan migrateコマンドが失敗するかもしれません。その場合は数分待ってください。

6. Laravel Scoutとthe MeiliSearch PHP SDKをインストールします。
```
docker run --rm -v $PWD:/app composer require laravel/scout meilisearch/meilisearch-php http-interop/http-factory-guzzle
```
7. コンフィグファイルを発行します。
```
docker-compose exec php sh
...
(after entering sh of php)
...
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```
8. Laravel\Scout\Searchableトレイトをuserモデル(app/Models/User.php)につけ加えます。 
```
(at the top level)
+use Laravel\Scout\Searchable;

(in the user class)
-use HasApiTokens, HasFactory, Notifiable;
+use HasApiTokens, HasFactory, Notifiable, Searchable;
```

9. テストユーザーレコードを作成します。

    (1) database/seeders/DatabaseSeeder.phpを設定します。
    ```
    -// \App\Models\User::factory(10)->create();
    +\App\Models\User::factory(10)->create();
    ```

    (2) シーダーを実行します。
    ```
    docker-compose exec php sh
    ...
    (after entering sh of php)
    ...
    php artisan db:seed
    ```
    注：
    手動でユーザーを作成することもできます。https://github.com/Asano-Naoki/docker_laravel_mailhog の前提条件６から９を参照。
    
    注：
    searchableトレイトをもつモデルデータは、作成時に自動的に検索可能になります。既存のデータ（searchableトレイトをもたずに作成されたデータ）を検索可能にしたい場合は、バッチインポートを実行してください。
    ```
    php artisan scout:import "App\Models\User"
    ```

エラーについては https://github.com/Asano-Naoki/docker_laravel_minimum も参照してください。


## Usage
検索可能なモデルデータを作成（またはバッチインポート）してください。http://localhost:7700/ でそれらを検索することができます。ポップアップウィンドウにはmeilisearchのマスターキー（サンプル.envファイルをコピーしたなら"masterKey"）を入力してください。

Laravelアプリケーションでmeilisearchを使うこともできます。詳しくは公式ドキュメントを読んでください。

ここにユーザーを検索する非常に単純な例を示します。

以下のコードの断片をroutes/web.phpに追記してください。
```
Route::get('user_search', function(){
    //htmlの検索フォームの表示
    echo <<<END
      <form method="get">
      <input name="search" value=""/>
      <input type="submit" value="Search">
      </form>
    END;
    
    //userモデルの取得
    if (isset($_GET['search'])) {
      $users = \App\Models\User::search($_GET['search'])->get();
    } else {
      $users = \App\Models\User::all();
    }
    
    //ユーザーの表示
    foreach ($users as $user) {
        echo '<li>'. $user->name .'</li>';
    }
});
```
http://localhost/user_search でユーザーを検索できます。

警告：
このコードの断片は一時的なテスト目的のためだけのものです。コントローラー、ビュー、検証などを使ってください。特に他の人が入力したデータをエスケープせずに使うことはしないでください。


## 著者
[浅野直樹](https://asanonaoki.com/blog/)


## ライセンス
MITライセンスの元にライセンスされています。詳細は[LICENSE](/LICENSE)をご覧ください。