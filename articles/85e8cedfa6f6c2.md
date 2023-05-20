---
title: "DBコンテナの起動を待ってアプリケーションのコンテナを起動したいのでヘルスチェックを追加する"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['docker', 'docker-compose', 'mysql']
published: true
---

# 概要

以下のコマンドを `make` コマンドで実行したところ、DBコンテナの起動が間に合わずにアプリケーションのコンテナが起動できないという問題が発生した。

```shell
$ nake up

# docker compose stop
# docker compose down --remove-orphans
# docker compose up -d
```

そのため、DBコンテナに対してヘルスチェックを行い、起動が完了してからアプリケーションのコンテナを起動するようにした。

# 解決策

`docker-compose.yml` に以下のように `healthcheck` を追加する。

```yaml
version: '3.8'
services:
  db:
    image: mysql:8.0.25
    container_name: db
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: app
      MYSQL_USER: app
      MYSQL_PASSWORD: app
    ports:
      - "3306:3306"
    healthcheck:
      test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]
      interval: 5s
      timeout: 3s
      retries: 30
  app:
    build:
      context: .
      dockerfile: ./docker/app/Dockerfile
    container_name: app
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      db:
        condition: service_healthy
```

# 結果

`make up` を実行したところ、DBコンテナの起動が完了してからアプリケーションのコンテナが起動するようになった。

```shell
$ make up

# docker compose stop
# docker compose down --remove-orphans
# docker compose up -d

Creating db ... Healthy
Creating app ... Started
```