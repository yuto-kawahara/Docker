## Docker とは

アプリを簡単に開発・デプロイできる仕組みのこと
Docker がないと・・・

- 環境構築が大変
- 人によっては動く動かないなどエラーが発生しやすい
  Docker のメリット
- 開発環境を簡単に用意
- メンバー間で開発環境の統一ができる
- テスト環境や本番環境の統一ができる

## Linux コマンド

- ls・・・現在のディレクトリのフォルダとファイル一覧を表示
- pwd・・・現在いるディレクトリの場所を表示
- cd・・・ディレクトリの移動
- mkdir・・・新規フォルダを作成
- touch・・・新規ファイルを作成
- echo・・・ファイルに入力した値を書き込む
- cat・・・ファイルの中身を表示
- less・・・全画面でファイルの中身を表示（ファイルの中身が長い場合に使用）
- mv・・・ファイルの位置を変える場合やファイル名を変更
- rm・・・ファイルやフォルダの削除

## Docker の仕組み

docker run hello-world の実行
イメージ・・・アプリケーションコードと OS が載った雛形
コンテナ・・・イメージを元にコンテナを作成、実行環境
デーモン・・・クライアントからコマンドを受け取るバッググラウンドのアプリケーション
受け取ったコマンドによって Docker レジストリにイメージを検索しに行き、ローカルに取得し、取得したイメージからコンテナを作成する

## Dockerfile の作成

イメージの雛形
docker image build コマンドで Dockerfile からイメージを作成する
docker container run

FROM・・・Docker のベースイメージを指定する
WROKDIR・・・自分の作業場所
COPY・・・アプリケーションコードを Docker にコピー

docker container run -it
-it・・・インタクラティブモードで Docker を起動させるためのオプション
-v・・・ローカルのフォルダと Docker のフォルダの共有を設定するためのオプション

## Docker Compose で Rails の開発環境を構築

Web サービスは複数のアプリケーションを使用して成り立っており、それを Docker で開発環境を構築しようとするとアプリケーション間の通信や設定が面倒
Docker Compose を使用することで複数のアプリケーションをまとめて操作することが可能
docker-compise.yml

### 基本操作

- イメージのビルド
  docker-compose build
- コンテナの作成と軌道
  docker-compose up -d(バッググラウンドで起動)
- コンテナを停止・削除
  docker-compose down

* コンテナの一覧を表示
  docker-compose ps
* ログを表示
  docker-compose logs
* コンテナを作成してコマンドを実行
  docker-compose run <サービス> <コマンド>
* 起動中のコンテナにコマンド実行
  docker-compose exec <サービス> <コマンド>

### フォルダ構成

プロジェクトのルートディレクトリに以下のファイルを配置

- docker-compose.yml
- Dockerfile
- src/rails の実行ファイルを配置

### Dokcerfile

FROM ruby:2.7
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
 && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
 && apt-get update -qq \
 && apt-get install -y nodejs yarn
WORKDIR /app
COPY ./src /app
RUN bundle config --local set path 'vendor/bundle' \
 && bundle install

RUN で必要なライブラリをインストールやアップデートを実施  
WORKDIR で作業ディレクトリを指定  
COPY で rails のソースコードを/app にコピーする  
RUN でインストール先を指定して bundle install を実行

docker-compose build・・・イメージを作成  
docker-compose run web rails new . --force --database=mysql  
docker-compose up（-d）・・・Docker の起動 -d オプションでバッググラウンドで起動

## Docker を本番環境で公開

１ 事前準備  
２ Heroku にログイン  
３ Heroku アプリを作成  
４ DB を追加・設定  
５ Dockerfile を本番環境用に修正  
６ Docker イメージをビルド・リリース  
７ 機能追加

### 事前準備

- github へアカウント登録
- heroku へアカウント登録
- heroku CLI のインストール

### Heroku へログイン

- iTerm から heroku login でログインを実行
- 何かしらのキーを押すかやっぱやめるかを選択する →Enter キー押す
- ブラウザでログイン認証画面が立ち上がるのでログイン実施

* heroku コンテナレジストリにコンテナのイメージをデプロイする
* heroku container:login

### heroku アプリの作成

- heroku create <アプリ名>で heroku 上で動くアプリを管理<アプリ名>は重複できないため一意の名前とする

* heroku addons:create cleardb:ignite -a rails-docker-shimeji でアプリケーションにアドオンを追加する

### DB を追加・設定

- heroku の config にデータベースの環境変数を追加
- rails アプリケーションの config の database.yml の production にデータベースアクセス用の設定を環境変数で設定する

### Docker ファイルを本番用に修正

ローカル環境でサーバーを実行して、アプリケーションがエラー無しで立ち上がることを確認した後に、heroku へビルドとコンテナレジストリに push する
その後、heroku release でデプロイし heroku open でアプリケーションを実行する

## Docker で CI/CD を構築

本番環境まで構築できたが、手動デプロイだと面倒・・・  
CI・・・継続的インテグレーション、ビルド・テストを自動実行  
CD・・・継続的デリバリー自動デプロイ  
CircleCI・・・CI/CD 構築サービス

### CI の設定

- テストコードの記載
- CircleCI に登録
- プロジェクトを登録
- config を設定
- 環境変数を設定
- Github にプッシュ
- テストを修正

docker image build -t dockerdemo ./
・・・Docker ファイルを元にイメージを作成するコマンド

## コンテナ入門

◆ アプリケーションを構成するコンポーネント

- ランタイム、エンジン
- コード
- ライブラリなどの依存物
- 設定

◆ 異なる複数の環境

- ローカル環境・・・自身の PC の開発環境
- アプリケーションコードは Git などの共有ストレージにアップロード
- ステージング/QA・・・テストする環境
- 本番環境・・・デプロイする環境

ローカルでは動いたけど、本番では動かない原因

- 環境の作成手順が違う
- ライブラリのバージョンが違う
- それを解決するのがコンテナという考え方、ラインタイム/エンジン・依存物・アプリケーションコードをパッケージングする＝コンテナ
- コンテナをそれぞれの環境にデリバリーすることで
