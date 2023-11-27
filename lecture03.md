# AWS第3回課題

### 課題①

- サンプルアプリケーションのデプロイ、ブラウザの確認

- 環境構築（rubyのバージョン、MySQLのボリューム拡張、設定）

### 課題②

- APサーバーについて調べる
- DBサーバーについて調べる
- Railsの構成管理ツールについて調べる

### 課題③

- 今回の課題から学んだことを報告

### 課題④

- GitHubでPR作成してURLを報告



#### 課題①

![2.アプリ起動ブラウザ_2023-11-26 0.25.55](https://github.com/yuji-oba/Garage/blob/main/1.%E3%82%A2%E3%83%95%E3%82%9A%E3%83%AA%E8%B5%B7%E5%8B%95_%202023-11-26%200.23.06.png)

サンプルアプリをブラウザで確認。

`GET` `POST`ができることも確認できた。



調べたこと

- `rvm` Rubyのバージョン切り替えソフトRuby Version Managerの略

  Rubyのインストール、アンインストールができる。

  複数のRubyのバージョンを管理できる

- `npm`Node.jsのパッケージ管理ツールNode Packege Manegerの略

  JavaScriptのパッケージの管理ツール

- `yarn` Node.jsのパッケージ管理ツールNode Packege Manegerの略

  JavaScriptのパッケージの管理ツール

  `yarn` をインストールするときは`npm install`で実行する。



実際にターミナルなどのコマンドと意味を簡単にまとめる

```bash
git clone https://github.com/yuta-ushijima/raisetech-live8-sample-app.git
`サンプルアプリのURL入手`
ruby -v
`rubyのバージョン確認`
ruby -h
`ruby usageの確認 コマンドのオプションが一覧で出る。さらに詳細を知るにはruby --help`
rvm install -v
`rvmのインストールされたバージョン確認`
rvm get stable
`rvmの最新版を入手`
rvm install 3.1.2
`Ruby3.1.2のインストール 開発環境に合わせる`
rvm use 3.1.2
`rubyのバージョンを3.1.2に切り替える`
ruby -v
`rubyのバージョン確認`

`DB MySQLの設定 課題補助教材の手順で`
EC2のEBSのボリュームを16GBに増強

curl -fsSL https://raw.githubusercontent.com/MasatoshiMizumoto/raisetech_documents/main/aws/scripts/mysql_amazon_linux_2.sh | sh
`scriptsフォルダのファイルを呼び出していて、yumのアップデート、mariadb削除、GPGキー更新が含まれる`

sudo cat /var/log/mysqld.log | grep "temporary password" | awk '{print $13}'
sudo cat /var/log/mysqld.log | grep "temporary password"
mysql -u root -p
`パスワードの設定とログイン`

ALTER USER 'root'@'localhost' IDENTIFIED BY '設定するパスワード';
FLUSH PRIVILEGES;
`後に初期パスワードではMySQLが繋げなかったのでパスワード変更`

cd raisetech-live8-sample-app
`アプリのディレクトリへ移動`

cp config/database.yml.sample config/database.yml
`sampleのymlをコピーしてdatabaseに使うymlを作成`

default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password:
`ymlファイルにMySQLに入るPassを設定`

node -v
v18.17.1
`nodeの確認`

nvm install v17.9.1
`開発環境のnodeのバージョンをインストール`

nvm use 17.9.1
`nodeのバージョン指定`

node -v
v17.9.1
`nodeバージョン確認`

npm install -g yarn
`yarnをグローバルインストール（-g PCのどのディレクトリからでもアクセスできる）`

yarn -v
1.22.21
`yarnのバージョン確認`

bin/cloud9_dev
bash: bin/cloud9_dev: Permission denied
`権限がないと拒否される`

ls -la bin
-rw-rw-r--  1 ec2-user ec2-user  158 Nov 25 09:27 cloud9_dev
`アクセス権限の確認`

sudo chmod 755 bin/cloud9_dev
`アクセス権を変更する`
`所有者の権限:読み込み,書き込み,実行`
`グループの権限:読み込み,実行`
`その他の権限:読み込み,実行`

ls -la bin
-rwxr-xr-x  1 ec2-user ec2-user  158 Nov 25 09:27 cloud9_dev
`アクセス権限変更の確認`

bin/cloud9_dev
`アプリケーションサーバーの起動`

previewでBlocked hostが出る

→railsファイルのconfig→envoiroments→deveropementt.rbに
config.hosts << "ebfb1635f2f14f2691aabe531301537d.vfs.cloud9.ap-northeast-1.amazonaws.com"
の文字列を最終行にコピー

bin/cloud9_dev
`サンプルアプリ起動`
`ブラウザで確認OK`
```



#### 課題②

#### サーバーについて

- APサーバー：puma
- バージョン：5.6.5

- APサーバーを終了した場合アクセスできるか否か？

  → できない※puma停止後のブラウザを確認

- 再度APサーバーを起動※`rails s`のコマンドでAPサーバー再起動



1. アプリを起動する`bin/cloud9_dev`

![1.アプリ起動_ 2023-11-26 0.23.06](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/1.アプリ起動_ 2023-11-26 0.23.06.png)

2.ブラウザの確認と `GET` `POST` されているか確認

![2.アプリ起動ブラウザ_2023-11-26 0.25.55](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/2.アプリ起動ブラウザ_2023-11-26 0.25.55.png)

3. APサーバー停止 `ctrl + c` → puma停止

![3.puma停止2023-11-26 0.28.34](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/3.puma停止2023-11-26 0.28.34.png)

4. APサーバーpuma停止後のブラウザ

![4.puma停止後のブラウザ_2023-11-26 0.30.31](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/4.puma停止後のブラウザ_2023-11-26 0.30.31.png)

5. APサーバーpuma再起動 `rails s`コマンドでアプリケーションのサーバーを起動

![](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/5.puma再起動_ 2023-11-26 0.31.58.png)

6. puma再起動後ブラウザ確認。`GET` `POST`共に問題ない。

![6.puma再起動後_2023-11-26 1.25.43](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/6.puma再起動後_2023-11-26 1.25.43.png)



#### DBサーバーについて

- DBサーバー：MySQL

- バージョン：8.0.35

  DBサーバー終了した場合、サイトにアクセスできるか否か？

- →できない。ターミナルとブラウザで`Can't connect to local MySQL server` のエラー確認



1. MySQLバージョン確認

- ローカル：`mysql --version`で確認

- MySQLログイン時に確認
- MySQLログイン後、`select version();` 、`status`で確認できる

![7.mysqlバージョン確認_ 2023-11-26 1.39.18](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/7.mysqlバージョン確認_ 2023-11-26 1.39.18.png)



2. DBサーバーの停止

APサーバーを停止して`sudo service mysqld stop`でDBサーバーを停止する。

`rails s` でAPサーバーを起動すると、ターミナルで`ActiveRecord::ConnectionNotEstablished (Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)):`MySQLに繋げないとのエラーログ

![8.DBサーバー停止後のAPサーバー再起動_ 2023-11-26 1.48.11](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/8.DBサーバー停止後のAPサーバー再起動_ 2023-11-26 1.48.11.png)



ブラウザでも`ActiveRecord::ConnectionNotEstablished`

`**Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)**`とエラーメッセージ

![](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/９.DBサーバー停止後のブラウザ画面_2023-11-26 1.46.55.png)



3. DBサーバーの状態確認とDBサーバーの再起動

`sudo service mysqld status`でMySQLの状態確認ができる。`inactive`確認

`sudo service mysqld status`でDBサーバーを再移動して動いているか再度確認

![10.MySQL状態確認と再起動_2023-11-26 2.09.29](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/10.MySQL状態確認と再起動_2023-11-26 2.09.29.png)



4. pumaを`rails s`で再起動後、ブラウザを確認。

   `GET` `POST`が機能していることも確認できた。

![11.DBサーバー再起動後の確認スクリーンショット 2023-11-26 2.21.57](/Users/Yuji/Desktop/RaiseTech_AWS/AWSフルコース第3回_提出課題/11.DBサーバー再起動後の確認スクリーンショット 2023-11-26 2.21.57.png)

#### Rubyの構成管理ツールについて

- 構成管理ツール：Bundler

  Gemを管理するツール

  Gemの依存関係とバージョンを管理するツール

  Bundlerにより、チーム全員のGemを統一したり、Gemのバージョンの切り替えが可能

  開発環境に合わせたGemのバージョンを合わせることが出来る



#### 課題③

課題で学んだこと

- Gitから入手したサンプルアプリをデプロイする一つの流れ
- DBのMySQLの設定、扱い方
- APサーバの扱い方
- Gem 構成管理ツール

サンプルアプリをデプロイしたが、必要なGemやツールのインストール、バージョンを合わせる、権限の設定変更など、多くあることがわかった。簡単なアプリだったからなんとかなったが、もっと複雑なアプリなどでは設定するだけでも、多くのツールやソフトのインストールが必要になるのだろうと感じた。

また、今回は講座内での手順を参考に進めたが、これがBundlerを使えば簡単に済むとのことなのだろうか？

調べてみたいし、試してみたい。

構成管理ツールはそれぞれの言語に複数あるのがわかった。しかも、メジャーか、使いやすいか、出来ることできない事など差もかなりある。

まだ、注意すべき部分がどこで、何かが分からないので時間もかかるし、労力かかるが、理解して数をこなせば楽にこなせるようになると信じてやっていきたい。

