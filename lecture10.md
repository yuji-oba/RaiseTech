# Lecture10

## CloudFormation
- CloudFormationで今までの講義で構築したAWS リソースをTemplateにコード化し、自動構築する。


### 構成図

#### CloudFormationで構築したリソースの構成図
![diagram_lecture10](images/lecture10/diagram_lecture10.jpg)


### Template

#### CloudFormationで構築したリソース

- スタックの作成は上記表の上から順に作成。
- Templateファイルは`cloudformation/lecture10`に格納

| Templateファイル | 構築されるリソース | 備考 |
| --------------- | -------------- | ---- |
| [cf_vpc_lecture.yml](cloudformation/lecture10/cf_vpc_lecture.yml) | VPC<br>Subnet<br>InternetGateway<br>RouteTable<br>Route | Subnetを`ap-northeast-1a`と`ap-northeast-1c`に配置し`InternetGateway`に接続するSubnetを`PublicSubnet`とする。 |
| [cf_securitygroup_lecture.yml](cloudformation/lecture10/cf_securitygroup_lecture.yml) | SecutiryGroup | 3つのセキュリティグループを構築<br> `EC2`のセキュリティグループ<br> `RDS`のセキュリティグループ<br> `ALB`のセキュリティグループ |
| [cf_iamrole_lecture.yml](cloudformation/lecture10/cf_iamrole_lecture.yml) | IAMRole | `S3`アクセス制限をアタッチする`IAMRole` |
| [cf_rds_lectrure.yml](cloudformation/lecture10/cf_rds_lectrure.yml) | DBInstance<br>DBSubnetGroup | `DB Engine`は`MySQL`<br>`DBSubnetGroup`は`PrivateSubnet`を指定 |
| [cf-ec2_lecture.yml](cloudformation/lecture10/cf-ec2_lecture.yml) | EC2Instance<br>KeyPair | `KeyPair`の値は`SystemManeger`>`パラメータストア`から取得 |
| [cf-alb_lecture.yml](cloudformation/lecture10/cf-alb_lecture.yml) | LoadBalancer | `ALB`を設定<br>`TargetGroup`を指定<br>`Listner`を定義 |
| [cf_s3_lecture.yml](cloudformation/lecture10/cf_s3_lecture.yml) | S3 Bucket | `パブリックアクセス`を全てブロック |



#### CloudFormationでのスタックの作成

Temlplateファイルをアップロードしてスタックを作成

![01_cf](images/lecture10/01_cf.jpg)



### 構築されたリソースの証跡


#### VPC

![02_cf](images/lecture10/02_cf.jpg)



##### Subnetの確認
- PublicSubnetとして構築したSubnetがInternetGatewayに繋がっているのを確認

![03_cf](images/lecture10/03_cf.jpg)

- PrivateSubnetとして構築したSubnetが他のネットワークに接続できないことを確認

![04_cf](images/lecture10/04_cf.jpg)

![05_cf](images/lecture10/05_cf.jpg)



#### SecurityGroup


##### EC2 SecurityGroup

インバウンドルール
- 自機PCからのSSH接続を許可
- HTTP接続、80port許可

![06_cf](images/lecture10/06_cf.jpg)

アウトバウンドルール

![07_cf](images/lecture10/07_cf.jpg)


##### RDS SecurityGroup

インバウンドルール
- ソースがEC2セキュリティグループの`sg-0f2d1f04109f4a192 / cf-securitygroup-EC2-lecture-ssh-http`である
- `port`が`3306`である

![08_cf](images/lecture10/08_cf.jpg)

アウトバウンドルール

![09_cf](images/lecture10/09_cf.jpg)


##### ALB SecurityGroup

インバウンドルール
- HTTP接続、80portを許可

![10_cf](images/lecture10/10_cf.jpg)

アウトバウンドルール

![11_cf](images/lecture10/11_cf.jpg)



#### IAMRole

AWSマネージドポリシーの`AmazonS3FullAccess`

![12_cf](images/lecture10/12_cf.jpg)



#### RDS
- エンジンは`MySQL`
- クラスは`db.t3.micro`
- AZは`ap-northeast-1a`

![13_cf](images/lecture10/13_cf.jpg)



#### EC2
- パブリックIPv4アドレスが付与されている
- `vpc-0c6f2fdc0837ded6d (CF-VPC-lecture)`に紐づいている
- `subnet-0e6aced4037a0d188 (CF-PublicSubnet-1a-lecture)`に紐づいている 
- 構築した`CF-IAMRole-S3FullAccess-lecture`が`IAMRole`としてアタッチされている

![14_cf](images/lecture10/14_cf.jpg)



##### EC2のセキュリティグループ
- 構築したEC2のセキュリティグループがアタッチされている

![15_cf](images/lecture10/15_cf.jpg)



#### ALB
- `vpc-0c6f2fdc0837ded6d (CF-VPC-lecture)`に紐づいている
- 構築したAZが2つ指定されている

![16_cf](images/lecture10/16_cf.jpg)

![17_cf](images/lecture10/17_cf.jpg)


- 構築したALB用のセキュリティグループがアタッチされている

![18_cf](images/lecture10/18_cf.jpg)

![19_cf](images/lecture10/19_cf.jpg)



#### S3

- パブリックアクセスが全てブロックされている

![20_cf](images/lecture10/20_cf.jpg)

![27_cf](images/lecture10/27_cf.jpg)



### 各リソースの接続とSecurityGroupの設定確認

#### EC2
- `Systems Manager`>`パラメーターストア`からkeyを取得し、SSH接続をする

![21_cf](images/lecture10/21_cf.jpg)


#### RDS
- MasterUserでのMySQLへログイン

![22_cf](images/lecture10/22_cf.jpg)


#### ロードバランサーの接続確認
- Nginxをインストールし、HTTP接続の確認
- `nginx.conf`ファイルのサーバー部分へALBのDNS名書き込み
- ALBのHealthCheckがHealthyになったのを確認してから`curl`コマンドでNginxのデフォルトページが返るのを確認

![28_cf](images/lecture10/28_cf.jpg)


- Webブラウザでの確認

![25_cf](images/lecture10/25_cf.jpg)
