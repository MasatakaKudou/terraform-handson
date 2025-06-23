# terraform-handson

ローカルでterraformを試そう

## 0. 事前準備

`docker compose` or `docker-compose` を使える

## 1. ハンズオン環境の準備

### 1-1. クローンする

`git clone git@github.com:MasatakaKudou/terraform-handson.git`

### 1-2. プロジェクトに移動しコンテナ起動

`docker compose up -d` or `docker-compose up -d`

以降は `docker compose` をベースに説明していく

### 1-3. 期待しているコンテナが立ち上がっているか確認

`docker compose ps`

以下コンテナが立ち上がる想定

| TH | TH |
| ---- | ---- |
| aws-cli | aws-cli実行環境 |
| localstack | 擬似AWS環境 |
| terraform | terraform実行環境 |

`STATUS` が全てUpになってればOK

## 2. Terraformを実行してみよう

### 2-1. terraform実行環境に入る

`docker compose exec terraform sh`

### 2-2. 初期化を行う

`terraform init`

### 2-3. 実行計画をプレビュー

`terraform plan`

## 3. S3バケットを作成してAWSに反映してみよう

### 3-1. terraformディレクトリ配下に `s3.tf`を作成し、以下を記述

```
resource "aws_s3_bucket" "s3-bucket" {
  bucket = "localstack-test-bucket"
}
```

### 3-2. 実行計画をプレビュー

`terraform plan`

### 3-3. AWS環境に反映

`terraform apply`

以下の出力があれば反映成功

`Apply complete! Resources: 1 added, 0 changed, 0 destroyed.`

## 4. S3バケットにファイルをアップロードしてみる

### 4-1. aws-cli実行環境に入る

`docker compose exec aws-cli sh`

### 4-2. アップロードする

`aws --endpoint-url=http://localhost:4566 s3 cp hello.png s3://localstack-test-bucket/`

### 4-3. アップロードされているか確認

以下のURLから確認

`http://localhost:4566/localstack-test-bucket/hello.html`
