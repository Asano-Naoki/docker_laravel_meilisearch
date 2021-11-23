[日本語版 README はこちら](/README_ja.md)

# docker_laravel_meilisearch
docker-compose for Laravel with meilisearch

## Summary
You can create minimum Laravel(LEMP) environment with meilisearch including a queue worker through docker-compose.

## Prerequisites
(I recommend you try https://github.com/Asano-Naoki/docker_laravel_minimum first.)

Before using this repository, prepare your Laravel project directory having models(databases) you'd like to search.

If you'd like to create a new one, follow the directions below:
1. Create a brand new Laravel project directory.
```
docker run --rm -v $PWD:/app composer create-project laravel/laravel example-meilisearch-app
```
2. Edit .env file.
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
3. Copy all the files and directory in this repository to the Laravel project directory with git command(fetch and merge) or manually.
```
git init
git add .
git commit -m "first commit"
git remote add origin git@github.com:Asano-Naoki/docker_laravel_meilisearch.git
git fetch
git merge origin/main --allow-unrelated-histories
```
4. Start docker-compose.
```
docker-compose up -d
```
5. Create users table.
```
docker-compose exec php sh
...
(after entering sh of php)
...
php artisan migrate
```
NOTE:
At the first setup, php artisan migrate command may fail because not all the mysql files are created. In this situation, you have just to wait for a few minutes.

6. Install Laravel Scout and the MeiliSearch PHP SDK.
```
docker run --rm -v $PWD:/app composer require laravel/scout meilisearch/meilisearch-php http-interop/http-factory-guzzle
```
7. Publish the configuration file.
```
docker-compose exec php sh
...
(after entering sh of php)
...
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```
8. Add the Laravel\Scout\Searchable trait to the user model. 

```app/Models/User.php
(at the top level)
+use Laravel\Scout\Searchable;

(in the user class)
-use HasApiTokens, HasFactory, Notifiable;
+use HasApiTokens, HasFactory, Notifiable, Searchable;
```

9. Create test user records.

    (1) Set the database/seeders/DatabaseSeeder.php
    ```
    -// \App\Models\User::factory(10)->create();
    +\App\Models\User::factory(10)->create();
    ```

    (2) Run the seeder.
    ```
    docker-compose exec php sh
    ...
    (after entering sh of php)
    ...
    php artisan db:seed
    ```
    NOTE:
    The model data with searchable trait become automatically searchable as soon as they are created. When you need to make existing data(the data created without searchable trait) searchable, you can do batch import.
    ```
    php artisan scout:import "App\Models\User"
    ```

See also https://github.com/Asano-Naoki/docker_laravel_minimum for possible errors.



## Usage
Create searchable model data(or do batch import) and you can search them at http://localhost:7700/. Enter meilisearch master key("masterKey" if you copy the .env file sample) in the popup window.

You can also use meilisearch in your Laravel application. Read the official documentation for details.

I'll show you very simple example to search users.

Append this code snippet to routes/web.php.

```
Route::get('user_search', function(){
    //rendering html form to search
    echo <<<END
      <form method="get">
      <input name="search" value=""/>
      <input type="submit" value="Search">
      </form>
    END;
    
    //retrieving a user model
    if (isset($_GET['search'])) {
      $users = \App\Models\User::search($_GET['search'])->get();
    } else {
      $users = \App\Models\User::all();
    }
    
    //showing users
    foreach ($users as $user) {
        echo '<li>'. $user->name .'</li>';
    }
});
```
You can search users at http://localhost:7700/user_search.

CAUTION:
This code is for only temporary testing purpose. You should use controllers, views, validations. Especially, don't use form data without escaping them.


## Author
[Asano Naoki](https://asanonaoki.com/blog/)


## License
Under the MIT License. See [LICENSE](/LICENSE) for details.