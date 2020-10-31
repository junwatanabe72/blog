---
title: dockerコマンド
date: "2020-10-31"
tags: ["docker"]
---

docker コマンドをいつも忘れてしまうので、
自分用のメモを残そう。

```shell
dockerコマンド
# dockerイメージを引張ってくるコマンド。
docker pull イメージ
# dockerイメージからコンテナを作成するコマンド。（停止状態。）
docker create イメージ
# docker コンテナを起動させるコマンド
docker start コンテナ
# pull + create + run を同時に実行するコマンド
docker run イメージorコンテナ
# docker内部でシェルを操作するコマンド(bashを使える場合。)
# itを指定しない場合、bashを操作できない。
docker exec -it コンテナ bin/bash(任意のコマンド)
# docker コンテナを停止させるコマンド
docker stop コンテナ
# docker コンテナを削除するコマンド
docker rm コンテナ

# Dockerfileからビルドするコマンド
# -tでタグ名を指定する。
docker build -t タグ名 パス


```

```shell
docker-composeコマンド

# docker-compose.ymlのあるディレクトリでdocker-composeを実行する
docker-compose up -d
#指定したコンテナのみ起動するコマンド
docker-compose start コンテナ

#指定したコンテナのみ停止するコマンド
docker-compose stop コンテナ
#起動している指定したコンテナのshellを操作するコマンド
docker-compose exec コンテナ bin/bash(任意のコマンド)
# docker-composeを削除するコマンド
docker-compose down

```
