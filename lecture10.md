# Lecture10

## CloudFormation
- CloudFormationで今までの講義で構築したAWS リソースをTemplateにコード化し、自動構築する。


### 構成図

#### CloudFormationで構築したリソースの構成図


### Template

#### CloudFormationで構築したリソース

- スタックの作成は上記表の上から順に作成。
- Templateファイルは`cloudformation/lecture10`に格納

| Templateファイル | 構築されるリソース | 備考 |
| --------------- | -------------- | ---- |
| ![cf_vpc_lecture.yml](cloudformation/lecture10/cf_vpc_lecture.yml) | VPC<br>Subnet<br>InternetGateway<br>RouteTable<br>Route | Subnetを`ap-northeast-1a`と`ap-northeast-1c`に配置し`InternetGateway`に接続するSubnetを`PublicSubnet`とする。 |
| ![cf_securitygroup_lecture.yml](cloudformation/lecture10/cf_securitygroup_lecture.yml) | SecutiryGroup | 3つのセキュリティグループを構築<br> `EC2`のセキュリティグループ<br> `RDS`のセキュリティグループ<br> `ALB`のセキュリティグループ |
| ![cf_iamrole_lecture.yml](cloudformation/lecture10/cf_iamrole_lecture.yml) | IAMRole | `S3`アクセス制限をアタッチする`IAMRole` |
| ![cf_rds_lectrure.yml](cloudformation/lecture10/cf_rds_lectrure.yml) | DBInstance<br>DBSubnetGroup | `DB Engine`は`MySQL`<br>`DBSubnetGroup`は`PrivateSubnet`を指定 |
| ![cf-ec2_lecture.yml](cloudformation/lecture10/cf-ec2_lecture.yml) | EC2Instance<br>KeyPair | `KeyPair`の値は`SystemManeger`>`パラメータストア`から取得 |
| ![cf-alb_lecture.yml](cloudformation/lecture10/cf-alb_lecture.yml) | LoadBalancer | `ALB`を設定<br>`TargetGroup`を指定<br>`Listner`を定義 |
| ![cf_s3_lecture.yml](cloudformation/lecture10/cf_s3_lecture.yml) | S3 Bucket | `パブリックアクセス`を全てブロック |



#### CloudFormation Templateファイル

##### cf_vpc_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for VPC Resources

Resources:
  CFVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: CF-VPC-lecture

# ----- subnetの定義 ----- #
# PublicSubnet ap-northeast-1aに作成
  CFPublicSubnet1a:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CFVPC
      CidrBlock: 10.0.0.0/20
      AvailabilityZone: "ap-northeast-1a"
      MapPublicIpOnLaunch: false #publicIPアドレスの設定
      Tags:
        - Key: Name
          Value: CF-PublicSubnet-1a-lecture

# PublicSubnet ap-northeast-1cに作成
  CFPublicSubnet1c:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CFVPC
      CidrBlock: 10.0.16.0/20
      AvailabilityZone: "ap-northeast-1c"
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: CF-PublicSubnet-1c-lecture

# PrivateSubnet ap-northeast-1aに作成
  CFPrivateSubnet1a:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CFVPC
      CidrBlock: 10.0.128.0/20
      AvailabilityZone: "ap-northeast-1a"
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: CF-PrivateSubnet-1a-lecture

# PrivateSubnet ap-northeast-1cに作成
  CFPrivateSubnet1c:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CFVPC
      CidrBlock: 10.0.144.0/20
      AvailabilityZone: "ap-northeast-1c"
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: CF-PrivateSubnet-1c-lecture

# ----- InternetGatewayの定義 ----- #
  CFInternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: CF-InternetGateway-lecture

  CFInternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId : !Ref CFInternetGateway
      VpcId: !Ref CFVPC

# ----- RouteTableの定義 ----- #
# PublicSubnet-1aとPublicSubnet-1cのRouteTable
  CFRouteTablePublicSubnet1a1c:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CFVPC
      Tags:
        - Key: Name
          Value: CF-RouteTable-PublicSubnet-1a-1c-lecture

# PrivateSubnet-1aのRouteTable
  CFRouteTablePrivateSubnet1a:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CFVPC
      Tags:
        - Key: Name
          Value: CF-RouteTable-PrivateSubnet-1a-lecture

# PrivateSubnet-1cのRouteTable
  CFRouteTablePrivateSubnet1c:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref CFVPC
      Tags:
        - Key: Name
          Value: CF-RouteTable-PrivateSubnet-1c-lecture

# ----- Routeの定義 ----- #
# PublicSubnet-1aとPublicSubnet-1cのRouteをInternetGatewayに繋げる
  CFRoutePublicSubnet:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref CFRouteTablePublicSubnet1a1c
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref CFInternetGateway

# ----- RouteTableとSubnetの紐付け定義 ----- #
# PublicSubnet-1a
  CFRouteTableAssociationPublicSubnet1a:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CFRouteTablePublicSubnet1a1c
      SubnetId: !Ref CFPublicSubnet1a

# PublicSubnet-1c
  CFRouteTableAssociationPublicSubnet1c:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CFRouteTablePublicSubnet1a1c
      SubnetId: !Ref CFPublicSubnet1c

# PrivateSubnet-1a
  CFRouteTableAssociationPrivateSubnet1a:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CFRouteTablePrivateSubnet1a
      SubnetId: !Ref CFPrivateSubnet1a

# PrivateSubnet-1c
  CFRouteTableAssociationPrivateSubnet1c:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref CFRouteTablePrivateSubnet1c
      SubnetId: !Ref CFPrivateSubnet1c

# ----- Outputs ----- #
Outputs:
  CFVPC:
    Description: VPC ID
    Value: !Ref CFVPC #!RefのReturnValuesはVPCのID 
    Export:
      Name: CF-VPC-ID #Name of resource to export

  CFPublicSubnet1a:
    Description: PublicSubnet ID located in ap-northeast-1a
    Value: !Ref CFPublicSubnet1a #!RefのReturnValuesはsubnetのID
    Export:
      Name: CF-PublicSubnet-1a-ID

  CFPublicSubnet1c:
    Description: PublicSubnet ID located in ap-northeast-1c
    Value: !Ref CFPublicSubnet1c #!RefのReturnValuesはsubnetのID
    Export:
      Name: CF-PublicSubnet-1c-ID

  CFPrivateSubnet1a: #RDSを配置するPrivateSubnet1aをoutput
    Description: PrivateSubnet ID located in ap-northeast-1a
    Value: !Ref CFPrivateSubnet1a #!RefのReturnValuesはsubnetのID
    Export:
      Name: CF-PrivateSubnet-1a-ID

  CFPrivateSubnet1c: #RDSを配置するSubnetGroupに必要なPrivateSubnet1cをoutput
    Description: PrivateSubnet ID located in ap-northeast-1c
    Value: !Ref CFPrivateSubnet1c #!RefのReturnValuesはsubnetのID
    Export:
      Name: CF-PrivateSubnet-1c-ID
```



##### cf_securitygroup_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for SecurityGroups Resources

Resources:
# ----- SecurityGroupの定義 ----- #
# EC2 自機MacでのSSH接続 80portへのHTTP接続を許可
  CFSecurityGroupEC2:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow http ssh access
      GroupName: cf-securitygroup-EC2-lecture-ssh-http
      SecurityGroupIngress: #インバウンドルール
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 106.73.168.225/32
          Description: Allow SSH access for my PC
        
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: Allow HTTP access
      Tags:
        - Key: Name
          Value: cf-securitygroup-EC2-lecture
      VpcId: !ImportValue CF-VPC-ID

# RDS EC2のアクセスを許可
  CFSecurityGroupRDS:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow RDS access for EC2
      GroupName: cf-securitygroup-RDS-lecture-admin
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref CFSecurityGroupEC2 #EC2のSecurityGroupのID参照
          Description: Allow db MySQL access for admin
      Tags:
        - Key: Name
          Value: cf-securitygroup-RDS-lecture
      VpcId: !ImportValue CF-VPC-ID

# ALB 80portへのHTTP接続を許可
  CFSecurityGroupALB:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow ALB access to EC2
      GroupName: cf-securitygroup-ALB-EC2
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: cf-securitygroup-ALB
      VpcId: !ImportValue CF-VPC-ID

# ----- Outputs ----- #
Outputs:
  CFSecurityGroupEC2:
    Description: SecurityGroup ID for EC2 #Information about the value
    Value: !Ref CFSecurityGroupEC2 #!RefのReturnValuesはSecurityGroupのID
    Export:
      Name: CF-SecurityGroup-EC2-ID

  CFSecurityGroupRDS:
    Description: SecurityGroup ID for RDS 
    Value: !Ref CFSecurityGroupRDS
    Export:
      Name: CF-SecurityGroup-RDS-ID

  CFSecurityGroupALB:
    Description: SecurityGroup ID for ALB
    Value: !Ref CFSecurityGroupALB
    Export:
      Name: CF-SecurityGroup-ALB-ID
```



##### cf_iamrole_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for IAMRole Resources

Resources:
# ----- IAMRoleの定義 ----- #
# S3のフルアクセス権限を付与 AWS管理ポリシーを指定
  CFS3IAMRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: CF-IAMRole-S3FullAccess-lecture
      Description: Allow S3 FullAccess for CFEC2
      AssumeRolePolicyDocument: #IAMRoleの信頼ポリシーの定義
        Version: 2012-10-17
        Statement: 
          - Effect: Allow
            Action: 
              - sts:AssumeRole
            Principal: 
              Service: 
                - ec2.amazonaws.com
      ManagedPolicyArns: # 管理ポリシーの指定
        - arn:aws:iam::aws:policy/AmazonS3FullAccess

  InstanceProfileS3FullAccess: 
    Type: AWS::IAM::InstanceProfile
    Properties:
      InstanceProfileName: !Ref CFS3IAMRole
      Roles: 
        - !Ref CFS3IAMRole

# ----- Outputs ----- #
Outputs:
  CFS3IAMRole:
    Description: IAMRole S3FullAccess RoleName 
    Value: !Ref InstanceProfileS3FullAccess
    Export:
      Name: CF-IamRole-S3FullAccess-RoleName
```



##### cf_rds_lectrure.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for RDS Resources

Resources:
# ----- RDSの定義 ----- #
# DBの設定 ap-northeast-1aに配置 MySQL 
  CFRDS:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage : 20
      AvailabilityZone: ap-northeast-1a
      DBInstanceClass: db.t3.micro
      Port: 3306
      StorageType: gp2 #ストレージタイプ 汎用SSD(gp2)
      BackupRetentionPeriod: 1
      MasterUsername: ***** # シークレット
      MasterUserPassword: ******** # パスワード シークレット
      DBInstanceIdentifier: CF-RDS-lecture
      Engine: mysql
      EngineVersion: 8.0.35 #latest MySQL Ver.
      DBSubnetGroupName: !Ref CFRDSSubnetGroup
      MultiAZ: false # マルチAZなし
      VPCSecurityGroups:
        - !ImportValue CF-SecurityGroup-RDS-ID

# SubnetGroupの設定 PrivateSubnet-1aとPrivateSubnet-1cを設定
  CFRDSSubnetGroup: #RDSを配置するPrivateSubnet-1aとPrivateSubnet-1cを指定
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: CF-RDS-SubnetGroup-1a-1c
      DBSubnetGroupName: CF-RDS-SubnetGroup-lecture
      SubnetIds:
        - !ImportValue CF-PrivateSubnet-1a-ID
        - !ImportValue CF-PrivateSubnet-1c-ID
```



##### cf-ec2_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for EC2 Resources

Resources:
# ----- KeyPairの定義 ----- #
  NewKeyPair:
    Type: AWS::EC2::KeyPair
    Properties:
      KeyName: CFEC2KeyPair

# ----- EC2の定義 ----- #
  CFEC2:
    Type: AWS::EC2::Instance #EC2Instanceの指定
    Properties:
      NetworkInterfaces:
        - AssociatePublicIpAddress: true # パブリックIPアドレスの自動割り当て
          SubnetId: !ImportValue CF-PublicSubnet-1a-ID #publicSubnet-1aに配置
          GroupSet:
            - !ImportValue CF-SecurityGroup-EC2-ID
          DeviceIndex: 0
      InstanceType: t2.micro # インスタンスのタイプ指定
      ImageId: ami-0556b98d8e7a269f1 #AMIの指定 Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type
      IamInstanceProfile: !ImportValue CF-IamRole-S3FullAccess-RoleName
      Tags:
        - Key: Name
          Value: CF-EC2-lecture
      KeyName: !Ref NewKeyPair

# ----- Outputs ----- #
Outputs:
  CFEC2:
    Description: EC2 Instance ID
    Value: !Ref CFEC2 #!RefのReturnValuesはInstanceのID
    Export:
      Name: CF-EC2-ID
```



##### cf-alb_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for ALB Resources

Resources:
# ----- ElasticLoadBalancingのApplicationLoadBalancerの定義 ----- #
# ALBを設定
  CFALB:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer #!RefはARN(AmazonResourceName)をreturn
    Properties:
      IpAddressType: ipv4
      Name: CF-ALB-lecture
      Scheme: internet-facing #ALBはinternet-facing
      SecurityGroups:
        - !ImportValue CF-SecurityGroup-ALB-ID
      Subnets:
        - !ImportValue CF-PublicSubnet-1a-ID
        - !ImportValue CF-PublicSubnet-1c-ID
      Tags:
        - Key: Name
          Value: CF-ALB-PublicSubnet-1a-1c

# ----- ALBのTargetGroupの定義 ----- #
  CFALBTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: CF-ALB-TargetGroup-lecture
      Port: 80
      Protocol: HTTP
      Targets:
        - Id: !ImportValue CF-EC2-ID
          Port: 80
      TargetType: instance
      VpcId: !ImportValue CF-VPC-ID

# ----- ALBのListenerの定義 ----- #
  CFALBListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      DefaultActions:
        - Type: forward
          TargetGroupArn:
            !Ref CFALBTargetGroup #!RefはARN(AmazonResourceName)をreturn
      LoadBalancerArn: !Ref CFALB #!RefはARN(AmazonResourceName)をreturn
      Port: 80
      Protocol: HTTP
```



##### cf_s3_lecture.yml
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: CloudFormation for S3 Resources

Resources:
# ----- S3の定義 ----- #
  CFS3:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: cf-s3-lecture #BucketNameに大文字は使用不可
      AccessControl: Private
      PublicAccessBlockConfiguration: #全てtrueにすることでパブリックアクセスを全てブロックの設定
        BlockPublicAcls: True
        BlockPublicPolicy: True
        IgnorePublicAcls: True
        RestrictPublicBuckets: True
      VersioningConfiguration:
        Status: Enabled

```



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
