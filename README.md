## 環境
PHP 8.0.15
Apache 2.4.52
mysql 8.0.20
Composer 2.2.6

## 前提
dockerのインストール、docker-desktopアプリをインストールしておきましょう。

## 導入手順
①git cloneする
git cloneしたいディレクトリに移動
例：projects配下に作成したい場合
projects % git clone https://github.com/kamoto0427/laravel-docker.git

↓
projects % ls
を実行し、laravel-dockerのディレクトリが作成されていることを確認する。

②ファイル名の変更する場合
mv 変更前のファイル名 変更後のファイル名でできます。
例：
projects % mv laravel-docker laravel-graphql

↓
projects % ls
ファイル名を変更したディレクトリがあることを確認する。
その後、変更したディレクトリに移動する。

projects % cd laravel-graphql
↓
laravel-graphql %

vsCodeを起動する。
laravel-graphql % code .

③.envファイルを作成し、.envファイルに環境変数を記載する。
.env_exampleに雛形書いているので、こちらをコピーし、.envを作成→環境変数を設定する
laravel-graphql % cp .env_example .env

以下のように環境変数を書く。docker-compose.ymlでは、${WEB_PORT}としているので、.envに書いた内容のポート番号が反映される。※まだ使っていないポート番号を記載すること。DB_PASSなどは任意
使っているポート番号の調べ方
参照：https://nordvpn.com/ja/blog/port-number/#:~:text=%E3%83%9D%E3%83%BC%E3%83%88%E7%95%AA%E5%8F%B7%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%92%E4%BD%BF%E3%81%86%E3%81%93%E3%81%A8%E3%81%AB%E3%82%88%E3%81%A3%E3%81%A6,%E7%95%AA%E5%8F%B7%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%97%E3%81%BE%E3%81%99%E3%80%82

ポート番号が既に使われていると、dockerコンテナ作成時に弾かれて作成できません。なので、使われていないポート番号にしてください。
例)
WEB_PORT=18111
DB_PORT=18222
DB_NAME=laravel_graphql_db
DB_ROOT_PASS=password
DB_PASS=secret
PMA_PORT=18333
PMA_USER=root
PMA_PASS=secret
PHP_CONTAINER=laravel_graphql_php
MYSQL_CONTAINER=laravel_graphql_mysql
PMA_CONTAINER=laravel_graphql_pma

PHP_CONTAINER、MYSQL_CONTAINER、PMA_CONTAINERはdockerのコンテナ名になります。

④dockerコンテナを作成&起動
laravel-graphql % docker-compose up -d

⑤コンテナが立ち上がったか確認
※docker-desktopのアプリからも確認できます。
!(https://github.com/kamoto0427/laravel-docker/tree/main/images/docker_ps実行.png)

⑥phpコンテナに入る
下記のdocker基本的な操作のphpコンテナに入るを参照。
または、vscodeでdockerの拡張機能を入れている場合は、もっと簡単にコンテナに入れます。
※vscodeの拡張機能でdockerと検索し、インストールする。
!(https://github.com/kamoto0427/laravel-docker/tree/main/images/vscodeからコンテナに入る.png)

⑦laravel9系をインストール
phpコンテナに入ったあと、以下を実行する。
/var/www# composer create-project --prefer-dist "laravel/laravel=9.*" .

"laravel/laravel=9.*" [ファイル名]ですが、"laravel/laravel=9.*" .と.を指定することでsrcのファイルが新たに作成され、そこにlaravelのソースが入ってきてくれます。  (docker-compose.ymlで.srcとvar/wwwを対応させているため。)
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
> @php artisan key:generate --ansi
Application key set successfully.
となれば成功です。

localhost:${WEB_PORT}
.envでWEB_PORTは18111にしていれば、
例)localhost:18111でlaravelの初期画面が出れば成功！

⑧laravel側の.envも修正する
src内にlaravelのソースが入っていますが、そこにも.envファイルがあるので、
DB_CONNECTION=mysql
DB_HOST=mysql # docker-compose.ymlのmysqlのサービス名(127.0.0.1だとエラー)
DB_PORT=3306 # ${DB_PORT}:3306とdocker-compose.ymlで設定しているが、${DB_PORT}の値だと接続できない
DB_DATABASE= # ${DB_NAME}で設定した値
DB_USERNAME= # ${DB_USER}で設定した値
DB_PASSWORD= # ${DB_PASS}で設定した値

⑨マイグレーションを実行する
phpコンテナに入り、マイグレーションを実行する
/var/www# php artisan migrate
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (183.87ms)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (137.12ms)
Migrating: 2019_08_19_000000_create_failed_jobs_table
Migrated:  2019_08_19_000000_create_failed_jobs_table (242.08ms)
Migrating: 2019_12_14_000001_create_personal_access_tokens_table
Migrated:  2019_12_14_000001_create_personal_access_tokens_table (332.70ms)
初期のマイグレーションが成功します。

## dockerの基本的な操作
* コンテナを作成するためにビルド、またはdockerファイルを修正してその変更を反映
laravel-docker % docker-compose up --build

* コンテナのイメージ、作成、起動するなら
laravel-docker % docker-compose up -d

* コンテナを起動
laravel-docker % docker-compose start

* コンテナを再起動
laravel-docker % docker-compose restart

* コンテナ停止
laravel-docker % docker-compose stop

* コンテナの確認
laravel-docker % docker ps
コンテナIDやコンテナ名、コンテナのステータスやポート番号などを確認できる

* phpコンテナに入る
% docker psでコンテナ情報を確認
CONTAINER ID IMAGE  COMMAND CREATED  STATUS PORTS  NAMESが確認できる

% docker exec -it <CONTAINER IDまたはNAMES> bash
例：
コンテナIDで入る場合
% docker exec -it 5443f829697c bash

NAMESで入る場合
% docker exec -it laravel_graphql_php bash

例)コンテナ名の場合
% docker exec -it laravel-docker_php_1 bash

* コンテナから抜ける
/var/www# exit

* mysqlコンテナに入る
% docker exec -it <CONTAINER IDまたはNAMES> bash

例)
% docker exec -it laravel-docker_mysql_1 bash
2d3e35:/#
