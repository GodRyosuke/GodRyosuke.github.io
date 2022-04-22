# Dockerのメモ
基本操作
このサイトも参考になる

https://qiita.com/wMETAw/items/34ba5c980e2a38e548db



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
