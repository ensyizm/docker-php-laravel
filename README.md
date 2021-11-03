# ローカルでPHP LaravelのDocker環境構築（macOS）

* 参考文献
https://qiita.com/ucan-lab/items/56c9dc3cf2e6762672f4

* 手順
"touch docker-compose.yml, mkdir -p infra/php, touch infra/php/Dockerfile, touch infra/php/php.ini"
"mkdir backend, docker compose up -d --build, docker compose ps, docker compose exec app bash,php -v, composer -V, php -m, exit"
"docker compose exec app php -v"でも確認できる
.
├── infra
│   └── nginx
│       └── default.conf # nginxの設定ファイル
├── backend
│  └── public # 動作確認用に作成
│       ├── index.html # HTML動作確認用
│       └── phpinfo.php # PHP動作確認用
└─── docker-compose.yml


"docker compose down, docker-compose.ymlにweb設定を追記, mkdir infra/nginx, touch infra/nginx/default.conf, docker compose up -d, docker compose ps, docker compose exec web nginx -v"
"mkdir backend/public, echo "Hello World" > backend/public/index.html, echo "<?php phpinfo();" > backend/public/phpinfo.php, rm -rf backend/*"
http://127.0.0.1:8080/index.html
http://127.0.0.1:8080/phpinfo.php

* Laravelをインストール
"feat laravel install"
[mac] $ docker compose exec app bash
[app] $ composer create-project --prefer-dist "laravel/laravel=8.*" .
[app] $ php artisan -V
Laravel Framework 8.40.0
[app] $ exit
http://127.0.0.1:8080/


* MySQLデータベースコンテナを作成
.
├── infra
│   └── mysql
│       ├── Dockerfile
│       └── my.cnf # MySQLの設定ファイル
└── docker-compose.yml

"docker compose down"
[mac] $ mkdir infra/mysql
[mac] $ touch infra/mysql/Dockerfile
[mac] $ touch infra/mysql/my.cnf
[mac] $ docker compose up -d
[mac] $ docker compose ps
            Name                          Command              State           Ports        
--------------------------------------------------------------------------------------------
docker-laravel-handson_app_1   docker-php-entrypoint php-fpm   Up      9000/tcp             
docker-laravel-handson_db_1    docker-entrypoint.sh mysqld     Up      3306/tcp, 33060/tcp  
docker-laravel-handson_web_1   nginx -g daemon off;            Up      0.0.0.0:8080->80/tcp

[mac] $ docker compose exec db mysql -V
mysql  Ver 8.0.24 for Linux on x86_64 (MySQL Community Server - GPL)
[mac] $ docker compose exec app bash
[app] $ php artisan migrate
    //エラー起こる
[app] $ exit

MySQLに接続拒否されたエラーなので、backend/.env のDB接続設定を修正する。
[mac] $ vim backend/.env
[mac] $ vim backend/.env.example
[mac] $ docker compose exec app bash
[app] $ php artisan migrate
//試しにデータ作成
[app] php artisan tinker
$user = new App\Models\User();
$user->name = 'phper';
$user->email = 'phper@example.com';
$user->password = Hash::make('secret');
$user->save();
Exit:  Ctrl+D

[mac] $ docker compose exec db bash
[db] $ mysql -u $MYSQL_USER -p$MYSQL_PASSWORD $MYSQL_DATABASE
[mysql] > SELECT * FROM users;
Exit:  Ctrl+D


# Docker環境の再構築手順
## Docker環境の破棄
コンテナの停止、ネットワーク・名前付きボリューム・コンテナイメージ、未定義コンテナを削除
[mac] $ docker-compose down --rmi all --volumes --remove-orphans

[mac] $ cd ..
[mac] $ rm -rf docker-laravel?

* GitHubからリポジトリをクローン
[mac] $ git clone git@github.com:@@@/docker-laravel.git
[mac] $ cd docker-laravel
[mac] $ docker compose up -d --build
http://127.0.0.1:8080/
/work/public/../vendor/autoload.php を開くのに失敗してエラーになっていることを確認

* app コンテナに入り、Laravelインストール
[mac] $ docker compose exec app bash
[app] $ composer install
[app] $ cp .env.example .env
[app] $ php artisan key:generate
[app] $ php artisan storage:link
[app] $ chmod -R 777 storage bootstrap/cache
http://127.0.0.1:8080/
[app] $ php artisan migrate
[app] $ exit
[mac] $ docker compose down

* MySQLクライアントツール「Sequel Ace」
ports:
  - 33060:3306
