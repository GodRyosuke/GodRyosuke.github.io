# Dockerのメモ
基本操作
このサイトも参考になる

[サイトへのリンク](https://qiita.com/wMETAw/items/34ba5c980e2a38e548db)



## imageをもらってくる
```terminal
docker pull ubuntu:18.04
```

##container 実行
docker  desktop->Containers/App
docker imageから新規container作成。
すでにコンテナがある場合は、以下のコマンドで開始できる。

#### 新しくコンテナを作る
```bash
docker run -it --name work ros:melodic # ros:melodic imageから workという名前のコンテナ作成
docker run -p 6080:80 ryosukekusumoto/ros-docker # ryosukekusumoto/ros-docker imageからport6080で実行
# -d:background activate -p:port number -v:<host dir>:<container dir> --name:container name
docker run -d -p 80:80 -v $(pwd)/nginx/log:/var/log/nginx --name webserver nginx
```

#### すでにあるコンテナを動かす

```bash
docker start wander_ros
```
#### 動いているコンテナに入る
```bash
docker container exec -it wander_ros_v2 bash # wander_ros_v2 containerに入る
```
#### コンテナ停止 & 削除　リスト
```bash
docker container stop wander_ros_v2 # container 停止
docker container rm wander_ros_v2 # container 削除
docker container list -a # 停止しているコンテナを含め、すべて表示
```
docker desktopにあるCIではないので注意!!


## dockerfileからimage作成
```Dockerfile:Dockerfile
FROM ubuntu:14.04
MAINTAINER Kate Smith <ksmith@example.com>
RUN apt-get update && apt-get install -y ruby rub
```
Ubuntu14.04と書いてあるが、docker pull ubuntu:14.04をしなくても動く。

```terminal
docker build -t godryosuke/getting-started:v3 .
```

## 既存のimageを改変して新しいimageを作成
対象としているimageからcontainerを作る
そのcontainerで作業

```terminal
docker commit -m "added ccc" -a "Tanaka Hanako" <container id> godryosuke/getting-started:v2

docker commit -m "Added json gem" -a "Kate Smith" \
0b2616b0e5a8 ouruser/sinatra:v2
```
imageができる。


## 作成したimageをdockerhubにpush
```terminal
docker push godryosuke/getting-started:v3
```

## dockerでros環境構築
基本以下のサイトでいける
https://qiita.com/yuki-shark/items/3e54af7140b755223c3e

以下のサイトも参照
https://qiita.com/SZZZUJg97M/items/dbdc784b92bde56cde3b

## windows上でrosを動かす
wander_ros_windows containerを起動
http://127.0.0.1:8000/ にアクセス
vscodeなどでcontainerアクセスもできる。

環境構築は、以下の通り。
https://qiita.com/karaage0703/items/957bdc7b4dabfc6639da
のチュートリアルにしたがって、tiryoのdocker image取得。
必要に応じて、普通のubuntu環境と同じようにsudo apt-get installしていく。
ユーザーがあったほうがいいので、上のやつでユーザー追加し、
su - wander
でログイン。


#docker compose
## wordpress環境構築
環境を構築したいディレクトリで
・wp-contentディレクトリ作成
以下のyamlファイルとDockerfileを作成する。

```terminal
wander_wp 
|- wp_content
|- docker-compose.yaml
|- Dockerfile
```


```yaml:docker-compose.yaml
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     build: .
     volumes:
       - ./wp-content:/var/www/html/wp-content # マウントに関する設定
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data: {}
```

```Dockerfile:Dockerfile
FROM wordpress:latest

ENTRYPOINT chown www-data:www-data /var/www/html/wp-content /var/www/html/wp-content/plugins /var/www/html/wp-content/uploads && docker-entrypoint.sh apache2-foreground
# chownで対象ディレクトリの所有者変更。
# docker-entrypoint.shでWordpressイメージのENTRYPOINT呼び出し。
```
以下のコマンド実行

```terminal
# 前のdocker composeを改変したときは、
docker-compose down // Docker Desktop のGUIでDownしてはだめ
docker-compose up -d
```
localhost:8000
でwordpress立ち上がる。
