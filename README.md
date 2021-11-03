# ローカルでPHP LaravelのDocker環境構築（macOS）

* 参考文献
https://qiita.com/ucan-lab/items/56c9dc3cf2e6762672f4

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
