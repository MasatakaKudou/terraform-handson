# terraform-handson

ローカル環境で Terraform を使って、疑似AWS環境（LocalStack）上に S3 バケットを作成・操作してみましょう

## 0. 事前準備

- docker compose（v2系）または docker-compose（v1系）が使える状態であること

## 1. ハンズオン環境の準備

### 1-1. リポジトリをクローン

```
git clone git@github.com:MasatakaKudou/terraform-handson.git
cd terraform-handson
```

### 1-2. コンテナを起動

```
docker compose up -d
```

※ `docker-compose` を使用してもOKです。以降は `docker compose` 表記で統一します

### 1-3. コンテナの起動確認

```
docker compose ps
```

| コンテナ名 | 用途 |
| ---- | ---- |
| aws-cli | aws-cli実行環境 |
| localstack | 擬似AWS環境 |
| terraform | terraform実行環境 |

全コンテナの `STATUS` が `Up` であることを確認。

### 1-4. ターミナルを2分割しておくと便利

- Terraform 操作用（`terraform` コンテナ）
- AWS CLI 操作用（`aws-cli` コンテナ）

※ すでに両方入っている前提のため、以降はコンテナへの `exec` コマンドは省略

## 2. Terraformを実行してみよう（terraformコンテナ）

### 2-1. 初期化を行う

```
terraform init
```

以下の出力があれば初期化完了！

`Terraform has been successfully initialized!`

### 2-2. 実行計画をプレビュー

```
terraform plan
```

以下の出力があればプラン成功！現時点ではリソース定義していないので変更なし

`No changes. Your infrastructure matches the configuration.`

## 3. S3バケットを作成してAWSに反映してみよう（terraformコンテナ）

### 3-1. カレントディレクトリに`s3.tf`を作成し、以下を記述

```hcl
resource "aws_s3_bucket" "s3-bucket" {
  bucket = "handson-bucket"
}
```

### 3-2. 実行計画を確認

```
terraform plan
```

反映しても問題ないか実行計画を確認

```hcl
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.s3-bucket will be created
  + resource "aws_s3_bucket" "s3-bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "handson-bucket"
      + bucket_domain_name          = (known after apply)
      + bucket_prefix               = (known after apply)
      + bucket_regional_domain_name = (known after apply)
      + force_destroy               = false
      + hosted_zone_id              = (known after apply)
      + id                          = (known after apply)
      + object_lock_enabled         = (known after apply)
      + policy                      = (known after apply)
      + region                      = (known after apply)
      + request_payer               = (known after apply)
      + tags_all                    = (known after apply)
      + website_domain              = (known after apply)
      + website_endpoint            = (known after apply)

      + cors_rule (known after apply)

      + grant (known after apply)

      + lifecycle_rule (known after apply)

      + logging (known after apply)

      + object_lock_configuration (known after apply)

      + replication_configuration (known after apply)

      + server_side_encryption_configuration (known after apply)

      + versioning (known after apply)

      + website (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

### 3-3. AWSに反映

```
terraform apply
```

以下の出力があれば反映成功

`Apply complete! Resources: 1 added, 0 changed, 0 destroyed.`

## 4. S3バケットにファイルをアップロードしてみる（aws-cliコンテナ）

### 4-1. HTMLをアップロード

```
aws --endpoint-url=http://localstack:4566 s3 cp hello.html s3://handson-bucket/
```

以下の出力があればアップロード成功

`upload: ./hello.html to s3://handson-bucket/hello.html`

### 4-3. アップロードされているか確認

以下のURLをブラウザから確認

```
http://localhost:4566/handson-bucket/hello.html
```

以下の画面が表示されればOK

![スクリーンショット 2025-06-23 9 48 32](https://github.com/user-attachments/assets/abd69ce1-321d-428c-9723-7cff469208eb)

## 5. S3バケットを変更してみよう（terraformコンテナ）

### 5-1. `s3.tf` を編集して、バケットにタグを追加

```hcl
resource "aws_s3_bucket" "s3-bucket" {
  bucket = "handson-bucket"

  tags = {
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}
```

### 5-2. 実行計画を確認

```
terraform plan
```

以下の出力を見て想定通りの実行計画か確認

```hcl
# aws_s3_bucket.s3-bucket will be updated in-place
~ resource "aws_s3_bucket" "s3-bucket" {
  ~ tags_all                    = {
    + "Environment" = "dev"
    + "ManagedBy"   = "terraform"
  }
}

Plan: 0 to add, 1 to change, 0 to destroy.
```

### 5-3. AWSに反映

```
terraform apply
```

以下の出力があれば反映成功

`Apply complete! Resources: 0 added, 1 changed, 0 destroyed.`

## 6. AWS上でS3バケットの変更を確認してみよう（aws-cliコンテナ）

バケットに紐づくタグを取得する

```
aws --endpoint-url=http://localstack:4566 s3api get-bucket-tagging --bucket handson-bucket
```

以下の出力があれば実際に反映されている

```
{
    "TagSet": [
        {
            "Key": "Environment",
            "Value": "dev"
        },
        {
            "Key": "ManagedBy",
            "Value": "terraform"
        }
    ]
}
```

## 7. S3バケットを削除してみよう（terraformコンテナ）

### 6-1. `s3.tf` をコメントアウト

```hcl
# resource "aws_s3_bucket" "s3-bucket" {
#   bucket = "handson-bucket"
# 
#   tags = {
#     Environment = "dev"
#     ManagedBy   = "terraform"
#   }
# }
```

### 6-2. 実行計画を確認

terraform実行環境に移動し、実行計画を確認

```
terraform plan
```

以下の出力を見て想定通りの実行計画か確認

```hcl
  # aws_s3_bucket.s3-bucket will be destroyed
  - resource "aws_s3_bucket" "s3-bucket" {
      - arn    = "arn:aws:s3:::handson-bucket" -> null
      ...
    }

Plan: 0 to add, 0 to change, 1 to destroy.
```

### 6-3. オブジェクトを削除

aws-cli実行環境に移動し、オブジェクトを削除

```
aws --endpoint-url=http://localstack:4566 s3 rm s3://localstack-test-bucket/ --recursive
```

もしかしたらこれも必要かも

```
aws --endpoint-url=http://localstack:4566 s3api delete-object --bucket handson-bucket --key hello.html
```

### 6-3. AWSに反映

terraform実行環境に移動し、AWSに反映

```
terraform apply
```

以下の出力があれば反映成功

`Apply complete! Resources: 0 added, 0 changed, 1 destroyed.`

### 6-4. AWS上で確認

aws-cli実行環境に移動し、バケット一覧を取得する

```
aws --endpoint-url=http://localstack:4566 s3api list-buckets
```

Bucketsに該当のバケットが存在しなければ実際に削除されている

```
{
    "Buckets": [],
    "Owner": {
        "DisplayName": "webfile",
        "ID": "1234567890"
    }
}
```

## 7. 掃除

### コンテナを停止

```
docker compose down
```

### プロジェクトを削除

```
rm -rf terraform-handson
```
