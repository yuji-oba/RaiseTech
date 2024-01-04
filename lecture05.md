### 第5回課題

#### AWSリソースの確認

#### VPC

vpc-014d5ac0a92a70701 / raisetech-lecture-vpc



#### EC2

- VPC ID
  - vpc-014d5ac0a92a70701 / raisetech-lecture-vpc

- サブネット ID
  - subnet-0694b05933b99fddf / raisetech-lecture-subnet-public1-ap-northeast-1a

- セキュリティグループ
  - sg-07ddb659618702bfd - ec2-rt-lecture-sg

#### RDS

- AZ
  - ap-northeast-1a

- VPC
  - raisetech-lecture-vpc (vpc-014d5ac0a92a70701)

- サブネットグループ
  - sbntg-db-mysql-rt-lecture

- サブネット
  - subnet-0c11603d97e4db85b
  - subnet-0db1f0f80f8a04693



#### EC2環境構築

- raisetech-live8-sample-appのREADMEから動作環境のソフトやgemなど確認

  動作環境

  - `ruby 3.1.2`
  - `Bundler 2.3.14`
  - `Rails 7.0.4`
  - `Node v17.9.1`
  - `yarn 1.22.19`

- サーバーアップデート

```bash
sudo yum update -y
```

- パッケージインストール

```bash
sudo yum -y install git make gcc-c++ patch libyaml-devel libffi-devel libicu-devel zlib-devel readline-devel libxml2-devel libxslt-devel ImageMagick ImageMagick-devel openssl-devel ruby ruby-devel
```

- nvm-Nodeインストール

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
```

```bash
 . ~/.nvm/nvm.sh
```

```bash
nvm install 17.9.1
```

- Ruby

```zsh
gpg2 --keyserver keyserver.ubuntu.com --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
```

- Ruby rvm

```
\curl -sSL https://get.rvm.io | bash -s stable --rails
```

```zsh
source ~/.rvm/scripts/rvm
```

```zsh
rvm install 3.1.2
```

```zsh
ruby --version
ruby 3.1.2p20 (2022-04-12 revision 4491bb740a) [x86_64-linux]
```

- Bundler

```zsh
gem install bundler -v 2.3.14
```

- rails

```zsh
gem install rails -v 7.0.4
```

- yarn

```
npm install -g yarn
```

- アプリのクローン

```zsh
git clone https://github.com/yuta-ushijima/raisetech-live8-sample-app.git
```

- database.ymlファイルを作る

```bash
cp config/database.yml.sample config/database.yml
```

- database.ymlのファイル編集

```zsh
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: **********
  
development:
  <<: *default
  database: raisetech_live8_sample_app_development
  socket: /var/lib/mysql/mysql.sock
```

- セットアップ

```zsh
bin/setup
```



- セキュリティグループ変更
  - 3000ポートの開放

![09_ec2](images/09_ec2.jpg)



- アプリを組み込みサーバーで公開 ec2-MySQL

```bash
bin/dev
```



- ブラウザでアクセス

  - ec2のパブリックIP`13.231.37.28`に`:3000`をつけてアクセス

  - ポストができていることも確認

  - EC2のMySQLに繋いでいる状態

    

![10_web](images/10_web.jpg)

![11_web](images/11_web.jpg)



- RDSのMy SQLをDBとして繋ぐ
  - `database.yml`を編集
  - RDSのエンドポイントに繋ぐ

```zsh
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: admin
  password: **********
  host: db-mysql-rt-lecture.cdtllrfhq8ji.ap-northeast-1.rds.amazonaws.com
```



- ブラウザで確認

![12_web](images/12_web.jpg)

- MySQLでDB確認

```zsh
mysql -u admin -p -h db-mysql-rt-lecture.cdtllrfhq8ji.ap-northeast-1.rds.amazonaws.com
```

```zsh
mysql> show databases;
+----------------------------------------+
| Database                               |
+----------------------------------------+
| information_schema                     |
| mysql                                  |
| performance_schema                     |
| raisetech_live8_sample_app_development |
| raisetech_live8_sample_app_test        |
| sys                                    |
+----------------------------------------+
6 rows in set (0.00 sec)
```

```zsh
mysql> use raisetech_live8_sample_app_development;
```

```zsh
mysql> select * from fruits;
+----+--------+-----------+----------------------------+----------------------------+
| id | name   | row_order | created_at                 | updated_at                 |
+----+--------+-----------+----------------------------+----------------------------+
|  1 | test01 |         0 | 2023-12-18 20:28:49.595609 | 2023-12-18 20:28:49.595609 |
+----+--------+-----------+----------------------------+----------------------------+
1 row in set (0.00 sec)
```

#### Unicorn単体でのアプリ起動

- `unicorn.rb`の記述変更
  - `listen 3000`を追記 `unicorn.sock`に繋ぐのをコメントアウト

```zsh
#listen '/home/ec2-user/raisetech-live8-sample-app/unicorn.sock'
pid    '/home/ec2-user/raisetech-live8-sample-app/unicorn.pid'
listen 3000 #テスト用3000ポート接続
```

- unicorn起動

```zsh
bundle exec unicorn -c config/unicorn.rb -D -E development
```

- ブラウザから3000ポートにアクセス

![03_unicorn](images/03_unicorn.jpg)

- DBへの登録も確認

![06_unicorn](images/06_unicorn.jpg)



#### Nginx

- Nginxを有効化

```zsh
sudo amazon-linux-extras install nginx1
```

- ` yum` コマンドでnginxをインストール

```
sudo yum -y install nginx
```

- サーバー起動

```zsh
sudo systemctl start nginx
```

- curlでnginx起動しているか確認

- 

```zsh
curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



- ブラウザでも確認

![06_nginx](images/06_nginx.jpg)



#### unicornとNginxの連携でDBに登録する

- `/etc/nginx/nginx.conf`マスターファイルの修正

```zsh
user ec2-user;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    upstream unicorn_ec2-rt-lecture {
        server unix:/home/ec2-user/raisetech-live8-sample-app/unicorn.sock;
        }
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80;
        listen       [::]:80;
        server_name  54.65.135.109;
        root         /home/ec2-user/raisetech-live8-sample-app/public
	try_files    $uri/index.html $uri /unicorn_ec2-rt-lecture

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

	location / {
            proxy_pass http://unicorn_ec2-rt-lecture;
	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   	    proxy_set_header Host $http_host;
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /500.html;
        location = /500.html {
        }
    }
}
```

- `/config/unicorn.rb`ソケット通信確認

```zsh
worker_processes Integer(ENV["WEB_CONCURRENCY"] || 3)
timeout 15
preload_app true

listen '/home/ec2-user/raisetech-live8-sample-app/unicorn.sock'
pid    '/home/ec2-user/raisetech-live8-sample-app/unicorn.pid'

```

- セキュリティグループ確認
  - 80ポートの開放

![10_nginx](images/10_nginx.jpg)

この状態でEC2のパブリックIPにアクセスするとblockedhostエラーが出る

![13_nginx](images/13_nginx.jpg)

- blockedhostエラーの解消

`config.hosts << "unicorn_ec2-rt-lecture"`を

`/home/ec2-user/raisetech-live8-sample-app/config/environments/development.rb`に追記



- ブラウザ表示 DB登録の確認
  - textのみでsave
  - DB登録OK

![14_nginx](images/14_nginx.jpg)

- ブラウザ表示 DB登録の確認
  - textと画像でsave
  - DB登録OK

![15_nginx](images/15_nginx.jpg)



#### ALBの作成とアタッチ

- アプリをデプロイするvpcにALBを作成

![03_alb](images/03_alb.jpg)

![04_alb](images/04_alb.jpg)

- 途中でターゲットグループを作成
  - インスタンスを`ec2-rt-lecture`指定
  - ポートは80

![11_alb](images/11_alb.jpg)

![12_alb](images/12_alb.jpg)

- ターゲットグループ指定

![14_alb](images/14_alb.jpg)

![17_alb](images/17_alb.jpg)

- nginx.conf内serverディレクティブでALBのDNS名を指定
  - DNS名：`alb-ec2-rt-lecture-sample-app-535061234.ap-northeast-1.elb.amazonaws.com`

```zsh
server {
        listen       80;
        listen       [::]:80;
        server_name  alb-ec2-rt-lecture-sample-app-535061234.ap-northeast-1.elb.amazonaws.com;
        root         /home/ec2-user/raisetech-live8-sample-app/public;
	try_files    $uri/index.html $uri /unicorn_ec2-rt-lecture
```



- サーバーを起動しALBのDNSにアクセスする
  - unicorn起動
  - nginx起動
  - ヘルスチェックが通るのを確認

![18_alb](images/18_alb.jpg)



- ALBのDNS名でブラウザからアクセス。
  - DNS 名：`alb-ec2-rt-lecture-sample-app-535061234.ap-northeast-1.elb.amazonaws.com`

![19_alb](images/19_alb.jpg)

blockedhostエラー



- blockedhostエラー解消
  - `/config/environments/development.rb` を修正
  - `config.hosts << "alb-ec2-rt-lecture-sample-app-535061234.ap-northeast-1.elb.amazonaws.com"`を追記



- サーバーを起動しALBのDNSにアクセスする
  - ブラウザから再度アクセス
  - DNS名：`alb-ec2-rt-lecture-sample-app-535061234.ap-northeast-1.elb.amazonaws.com`



- ブラウザの表示

![20_alb](images/20_alb.jpg)

- textとimageが登録されるのを確認

![23_alb](images/23_alb.jpg)



#### S3画像を保存する

- バケットの作成
  - バケット名:ec2-rt-lecture-raisetech-live8-sample-app

![07_s3](images/07_s3.jpg)



- S3にIAMロールをアタッチ
  - S3Fullaccessを許可してIAMロールを作成

![18_s3](images/18_s3.jpg)



- EC2インスタンスにIAMロールをアタッチ

![19_s3](images/19_s3.jpg)



- EC2インスタンスにIAMロールが追記されたのを確認

![20_s3](images/20_s3.jpg)



- storage.ymlの修正
  - `/config/storage.yml`Amazonのbucketに追記

```zzsh
amazon:
 service: S3
 access_key_id: <%= Rails.application.credentials.dig(:aws, :access_key_id) %>
 secret_access_key: <%= Rails.application.credentials.dig(:aws, :secret_access_key) %>
 region: ap-northeast-1
 bucket: ec2-rt-lecture-raisetech-live8-sample-app
```



- development.rbの修正
  - `/config/environments/development.rb`を修正
  - `active_storage.service`の保存先を`local`から`amazon`に変更

```zsh
config.active_storage.service = :amazon
```



- ブラウザにアクセスしてS3への保存を確かめる

画像を登録

![23_s3](images/23_s3.jpg)

![24_s3](images/24_s3.jpg)

- 登録は成功

![25_s3](images/25_s3.jpg)


- マネジメントコンソールから画像が格納されていること確認

![26_s3](images/26_s3.jpg)