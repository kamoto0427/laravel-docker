## 環境
PHP 8.0.15
Apache 2.4.52
mysql 8.0.20
Composer 2.2.6
Laravel9

## 前提
dockerのインストール、docker-desktopアプリをインストールしておきましょう。

## 導入手順
### ①git cloneする<br>
git cloneしたいディレクトリに移動<br>
例：projects配下に作成したい場合<br>
```
projects % git clone https://github.com/kamoto0427/laravel-docker.git
```

↓<br>
```
projects % ls
```

を実行し、laravel-dockerのディレクトリが作成されていることを確認する。

### ②ファイル名の変更する場合<br>
mv 変更前のファイル名 変更後のファイル名でできます。<br>
例：<br>
```
projects % mv laravel-docker laravel-graphql
```

↓<br>
```
projects % ls
```
ファイル名を変更したディレクトリがあることを確認する。<br>
その後、変更したディレクトリに移動する。<br>

```
projects % cd laravel-graphql
```
↓<br>
```
laravel-graphql %
```

vsCodeを起動する。<br>
```
laravel-graphql % code .
```

### ③.envファイルを作成し、.envファイルに環境変数を記載する。<br>
.env_exampleに雛形書いているので、こちらをコピーし、.envを作成→環境変数を設定する。<br>
```
laravel-graphql % cp .env_example .env
```

以下のように環境変数を書く。<br>
docker-compose.ymlでは、${WEB_PORT}としているので、.envに書いた内容のポート番号が反映される。<br>
※まだ使っていないポート番号を記載すること。DB_PASSなどは任意<br>
使っているポート番号の調べ方<br>
参照：https://nordvpn.com/ja/blog/port-number/#:~:text=%E3%83%9D%E3%83%BC%E3%83%88%E7%95%AA%E5%8F%B7%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%99%E3%82%8B%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E3%82%92%E4%BD%BF%E3%81%86%E3%81%93%E3%81%A8%E3%81%AB%E3%82%88%E3%81%A3%E3%81%A6,%E7%95%AA%E5%8F%B7%E3%82%92%E7%A2%BA%E8%AA%8D%E3%81%97%E3%81%BE%E3%81%99%E3%80%82

ポート番号が既に使われていると、dockerコンテナ作成時に弾かれて作成できません。<br>
なので、使われていないポート番号にしてください。<br>
また、パスワードやユーザー名などはセキュリティレベルを上げてください。<br>
例)<br>
```
WEB_PORT=8111
DB_PORT=8222
MYSQL_DATABASE=laravel_graphql_db
MYSQL_ROOT_PASSWORD=Password1
MYSQL_USER=laravel_grapjql_user
MYSQL_PASSWORD=Password2
PMA_PORT=8333
PMA_USER=laravel_graphql_pma
PMA_PASS=Password3
```

### ④dockerコンテナを作成&起動<br>
```
laravel-graphql % docker-compose up -d --build
```

### ⑤コンテナが立ち上がったか確認<br>
※docker-desktopのアプリからも確認できます。<br>
<img width="475" alt="スクリーンショット 2022-11-13 21 23 03" src="https://user-images.githubusercontent.com/64408070/204076923-50615402-71db-489d-8b78-306cf8e6bfc7.png">

### ⑥phpコンテナに入る<br>
下記のdocker基本的な操作のphpコンテナに入るを参照。<br>
または、vscodeでdockerの拡張機能を入れている場合は、もっと簡単にコンテナに入れます。<br>
※vscodeの拡張機能でdockerと検索し、インストールする。<br>
<img width="333" alt="スクリーンショット 2022-11-26 16 14 29" src="https://user-images.githubusercontent.com/64408070/204077034-906f4974-4c80-4bf2-b63b-40a884ed5d14.png">

### ⑦laravel9系をインストール<br>
phpコンテナに入ったあと、以下を実行する。<br>
```
/var/www# composer create-project --prefer-dist "laravel/laravel=9.*" .
```

"laravel/laravel=9.*" [ファイル名]ですが、"laravel/laravel=9.*" .と.を指定することでsrcのファイルが新たに作成され、そこにlaravelのソースが入ってきてくれます。 <br>
※docker-compose.ymlで.srcとvar/wwwを対応させているため。<br>
```
> @php artisan vendor:publish --tag=laravel-assets --ansi --force
No publishable resources for tag [laravel-assets].
Publishing complete.
> @php artisan key:generate --ansi
Application key set successfully.
```
となれば成功です。<br>

.envでWEB_PORTは8111にしていれば、<br>
localhost:8111でlaravelの初期画面が出れば成功！<br>

### ⑧laravel側の.envも修正する<br>
src内にlaravelのソースが入っていますが、そこにも.envファイルがあるので、<br>
```
DB_CONNECTION=mysql
DB_HOST=mysql # docker-compose.ymlのmysqlのサービス名(127.0.0.1だとエラー)
DB_PORT=3306 # ${DB_PORT}:3306とdocker-compose.ymlで設定しているが、${DB_PORT}の値だと接続できない
DB_DATABASE= # ${DB_NAME}で設定した値
DB_USERNAME= # ${DB_USER}で設定した値
DB_PASSWORD= # ${DB_PASS}で設定した値
```

### ⑨マイグレーションを実行する<br>
phpコンテナに入り、マイグレーションを実行する<br>
```
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
```
初期のマイグレーションが成功します。

## dockerの基本的な操作
* コンテナを作成するためにビルド、またはdockerファイルを修正してその変更を反映<br>
```
laravel-docker % docker-compose up --build
```

* コンテナのイメージ、作成、起動するなら<br>
```
laravel-docker % docker-compose up -d
```

* コンテナを起動<br>
```
laravel-docker % docker-compose start
```

* コンテナを再起動<br>
```
laravel-docker % docker-compose restart
```

* コンテナ停止<br>
```
laravel-docker % docker-compose stop
```

* コンテナの確認<br>
```
laravel-docker % docker ps
```
コンテナIDやコンテナ名、コンテナのステータスやポート番号などを確認できる<br>

* phpコンテナに入る<br>
```
% docker ps
```
でコンテナ情報を確認<br>
CONTAINER ID IMAGE  COMMAND CREATED  STATUS PORTS  NAMESが確認できる<br>

```
% docker exec -it <CONTAINER IDまたはNAMES> bash
```
例：<br>
コンテナIDで入る場合<br>
```
% docker exec -it 5443f829697c bash
```

NAMESで入る場合<br>
```
% docker exec -it laravel_graphql_php bash
```

例)コンテナ名の場合<br>
```
% docker exec -it laravel-docker_php_1 bash
```

* コンテナから抜ける<br>
```
/var/www# exit
```

* mysqlコンテナに入る<br>
```
% docker exec -it <CONTAINER IDまたはNAMES> bash
```

例)<br>
```
% docker exec -it laravel-docker_mysql_1 bash
2d3e35:/#
```
