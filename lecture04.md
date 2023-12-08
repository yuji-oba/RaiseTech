### 第4回課題



#### VPC構築

![vpc_01_create_20231209](/images/lecture04/vpc_01_create_20231209.png)



#### EC2構築

![ec2_01_create_20231209](/images/lecture04/ec2_01_create_20231209.png)



#### RDS構築

![rds_01_create_20231209](/images/lecture04/rds_01_create_20231209.png)



#### Security Groups

##### EC2

![ec2_02_sg_20231209](/images/lecture04/ec2_02_sg_20231209.png)



##### RDS

![rds_02_sg_20231209](/images/lecture04/rds_02_sg_20231209.png)

###### インバウンドルール

![rds_03_sginbound_20231209](/images/lecture04/rds_03_sginbound_20231209.png)

アウトバウンドルール

![rds_04_sgoutbound_20231209](/RaiseTech/images/lecture04/rds_04_sgoutbound_20231209.png)



#### EC2からRDSへ接続

##### EC2への接続

```bash
ssh -i ~/.ssh/ec2-rt-lecture04.pem ec2-user@54.250.82.187
```

![ec2-rds_01_ec2connect_20231209](/images/lecture04/ec2-rds_01_ec2connect_20231209.png)

##### EC2からMySQLへの接続

```bash
mysql -u admin_rtlec04 -p -h db-rt-lecture04.cdtllrfhq8ji.ap-northeast-1.rds.amazonaws.com
```

![ec2-rds_02_mysqlconnect_20231209](/images/lecture04/ec2-rds_02_mysqlconnect_20231209.png)



##### 今回の課題で学んだ事

- VPC EC2 RDSの初歩的な構築方法
- セキュリティグループ　インバウンド、アウトバンドルール
- EC2からRDSへの接続
- 自分が構築するネットワークの配置をしっかりイメージしてVPC、EC2、RDS、セキュリティグループを構築することが重要。
- lecture04の資料にも記載あったが、**「ネットワークの箱がどう区切られていて、どういうアクセスができるのかイメージを脳内で描けるかどうかが、ひとつのポイントになってきます」**との言葉は常に意識していきたい。