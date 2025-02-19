## 概要

Dockerを使ったRuby on Railsの開発環境を構築するためのテンプレートリポジトリ。Railsアプリの構成は、以下の通り。
- Ruby：3.3.6
- Rails：7.2.2
- Node.js：20系
- JavaScriptバンドラー：ESBuild
- CSSフレームワーク：TailwindCSS
- データベース：PostgreSQL

## 環境構築の手順

このリポジトリの```Use this template```というボタンをクリック。

リポジトリの作成画面に遷移するので、アプリのリポジトリ名を入力してリポジトリを作成する。

作成したリポジトリを```git clone```して、ローカルに持ってくる。

ローカルのターミナルにて、以下のコマンドを1行ずつ実行する。

```
$ docker compose build
$ docker compose run --rm web gem install rails -v 7.2.2
$ docker compose run --rm web rails new . -d postgresql -j esbuild --css=tailwind --skip-kamal --skip-solid
```

```rails new```で作成されたファイルのうち、```config/database.yml```を以下のように編集する。

```
# デフォルト設定（共通設定）
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # https://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db # compose.ymlで定義したDBのコンテナ名
  username: postgres  # 接続ユーザー名：compose.ymlで定義したDBのユーザー名と一致
  password: password  # 接続パスワード：compose.ymlで定義したDBのパスワードと一致
```

```Procfile.dev```を以下のように編集する。

```
# Webサーバの起動設定
web: env RUBY_DEBUG_OPEN=true bin/rails server -b 0.0.0.0 -p 3000
# JavaScriptのビルド設定
js: yarn build --watch
# CSSのビルド設定
css: yarn build:css --watch
```

以下の```Docker compose```コマンドを実行して、コンテナを起動する。

```
$ docker compose up
```

コンテナの起動を確認したら、以下のURLにアクセスして、Railsの初期画面が表示されるか確認する。

http://localhost:3000/

## ```tailwindcss: not found```というエラーが出てTailwind CSSが反映されない場合

バージョン4系を利用したい場合、以下のQiita記事を参考のこと。
https://qiita.com/topi_log/items/b8cc6afaa6e12599ffbb

バージョン3系を利用したい場合、以下の記事を参考のこと。
https://qiita.com/YamzknA/items/53478370761f716b068f