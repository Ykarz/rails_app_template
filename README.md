## 概要

Dockerを使ったRuby on Railsの開発環境を構築するためのテンプレートリポジトリ。Railsアプリの構成は、以下の通り。
- Ruby：3.3.6
- Rails：7.2.2
- Node.js：20系
- JavaScriptバンドラー：ESBuild
- CSSフレームワーク：Tailwind CSS 3系
- UIコンポーネント：daisyUI
- データベース：PostgreSQL

## 環境構築の手順

このリポジトリの```Use this template```というボタンをクリック。

リポジトリの作成画面に遷移するので、アプリのリポジトリ名を入力してリポジトリを作成する。

作成したリポジトリを```git clone```して、ローカルに持ってくる。

ローカルのターミナルにて、以下のコマンドを1行ずつ実行する。
なお、Tailwind CSSのバージョンは3系で固定したいので、```--css=tailwind```オプションは使用しない。

```
$ docker compose build
$ docker compose run --rm web gem install rails -v 7.2.2
$ docker compose run --rm web rails new . -d postgresql -j esbuild --skip-kamal --skip-solid
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
css: bin/rails tailwindcss:watch
```

ここからTailwindをgem形式でインストールしていく。```Gemfile```に以下を記述する。

```
gem "tailwindcss-rails"
gem "tailwindcss-ruby", "3.4.17"  # "3.4.17"の部分がTailwindのバージョン
```

以下のコマンドを実行して、```Gemfile```に記述されたgemをインストールし、その後、RailsアプリにTailwindを反映させる。

```
$ docker compose run --rm web bundle install
$ docker compose run --rm web bin/rails tailwindcss:install
```

daisyUIをインストールする。以下のコマンドを実行する。

```
$ docker compose run --rm web yarn add -D daisyui@4
$ docker compose run --rm web yarn add postcss  # 上記のdaisyUIのインストールコマンドだけではダメみたい
```

Tailwindの設定ファイルである```config/tailwind.config.js```に、プラグインとしてdaisyUIを記述する。

```
const defaultTheme = require('tailwindcss/defaultTheme')

module.exports = {
  content: [
    './public/*.html',
    './app/helpers/**/*.rb',
    './app/javascript/**/*.js',
    './app/views/**/*.{erb,haml,html,slim}'
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter var', ...defaultTheme.fontFamily.sans],
      },
    },
  },
  plugins: [
    // 以下の部分がdaisyUIに関する記述
    require('daisyui'),
    // require('@tailwindcss/forms'),
    // require('@tailwindcss/typography'),
    // require('@tailwindcss/container-queries'),
  ]
}

```

以下の```Docker compose```コマンドを実行して、コンテナを起動する。
一応、ログを確認して、CSSのビルド等でエラーを吐いてないか確認しておくこと。

```
$ docker compose up
```

コンテナの起動を確認したら、以下のURLにアクセスして、Railsの初期画面が表示されるか確認する。

http://localhost:3000/