# Docker, UbuntuベースのLAMP環境でLaravel queueを実行

DockerにてUbuntuをベースにしたLAMP環境を構築し、Laravelのqueueを実行する手順です。

Windows WSL2にて実験しています。

基本的にはMacでも同様に実行できると思いますが、M1Macをご利用の場合、`docker-compose.yml`や`Dockerfile`に手を加える必要があるかもしれません。


## 開発環境

- Apache2
- PHP7.4
- Laravel8
- MariaDB

MariaDBはqueueを`jobs`テーブルに溜めているだけなので、特にバージョンの指定はありません。


## 環境構築手順

### Dockerのビルド・起動

Docker関連のファイルは`docker`ディレクトリ内にまとめています。

```
cd docker
docker-compose build
docker-compose up -d
```

### Laravelインストール

Laravelはubuntuコンテナ内に展開しているので、コンテナへアクセスします。

```
docker-compose exec ubuntu bash
```

以下の作業はubuntuコンテナ内で実行してください。

composerを使ってLaravel本体をインストールします。

```
composer install
```

その後、`.env.example`を`.env`としてコピーし、キャッシュをクリア、migrationを実行してください。

```
cp .env.example .env
php artisan optimize:clear
php artisan config:cache
php artisan migrate
```

`http://localhost`でアクセスして、Laravelの初期画面が表示されていれば、OKです。

## queueのテスト

queueのテストを行うためにログを書き出すだけの簡単なJobを追加しています。

以下のコマンドでJobが実行されます。

```
php artisan job:test
```

queueが実行されていれば、`storage/logs/laravel.log`に以下のような内容が書き込まれます。

```
[2022-02-19 12:24:40] local.DEBUG: TestJob is success!(2022-02-19 12:24:40)  
```

何度かコマンドを試してみて、ログが書き込まれていれば正常にqueueが実行されています。

### queueの注意点

`.env`ファイルの`QUEUE_CONNECTION`が`sync`になっていると、supervisorを介さずに直接実行されてしまいます。

そのため、以下のようになっていることを確認してください。

```
QUEUE_CONNECTION=database
```

## Dockerでsupervisorを動かすポイント

まだ知見が不十分のため、完全に正しいとは言えない部分もあります。ご了承ください。

### supervisord.confの設定

`supervisord.conf`を`docker/supervisor/supervisord.conf`の内容に入れ替える必要がありました。

具体的な変更箇所は以下のとおりです。

```
[unix_http_server]
file=/dev/shm/supervisor.sock   ; (the path to the socket file)
chmod=0700                       ; sockef file mode (default 0700)

※/var/run/... から /dev/shm/... に変更しています。
```

```
[supervisord]
nodaemon=true
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/dev/shm/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor            ; ('AUTO' child log dir, default $TEMP)

※nodaemon=true を追加しています。
※/var/run/... から /dev/shm/... に変更しています。
```

```
[supervisorctl]
serverurl=unix:///dev/shm/supervisor.sock ; use a unix:// URL  for a unix socket

※/var/run/... から /dev/shm/... に変更しています。
```

### laravel-worker.confの追加

`docker/supervisor/laravel-worker.conf`の内容を参照してください。

### docker-compose時のentrypointでsupervisorを実行

supervisorを実行するために`docker/docker-compose.yml`のubuntuコンテナにentrypointを設定しています。

```
entrypoint: "supervisord -c /etc/supervisor/supervisord.conf"
```

この記述により、Dockerを起動するたびにsupervisorも起動されるようにしています。