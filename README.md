# terraform-handson

ローカルでterraformを試そう

## 0. 事前準備

`docker compose` or `docker-compose` を使える

## 1. ハンズオン環境の準備

### 1-1. クローンする

```
git clone git@github.com:MasatakaKudou/terraform-handson.git
```

### 1-2. プロジェクトに移動しコンテナ起動

```
docker compose up -d
```

or

```
docker-compose up -d
```

以降は `docker compose` をベースに説明していく

### 1-3. 期待しているコンテナが立ち上がっているか確認

```
docker compose ps
```

以下コンテナが立ち上がる想定

| コンテナ名 | 用途 |
| ---- | ---- |
| aws-cli | aws-cli実行環境 |
| localstack | 擬似AWS環境 |
| terraform | terraform実行環境 |

`STATUS` が全てUpになってればOK

## 2. Terraformを実行してみよう

### 2-1. terraform実行環境に入る

```
docker compose exec terraform sh
```

### 2-2. 初期化を行う

```
terraform init
```

以下の出力があれば初期化完了！

`Terraform has been successfully initialized!`

### 2-3. 実行計画をプレビュー

```
terraform plan
```

以下の出力があればプラン成功！現時点ではリソース定義していないので変更なし

`No changes. Your infrastructure matches the configuration.`

## 3. S3バケットを作成してAWSに反映してみよう

### 3-1. カレントディレクトリに`s3.tf`を作成し、以下を記述

```hcl
resource "aws_s3_bucket" "s3-bucket" {
  bucket = "localstack-test-bucket"
}
```

### 3-2. 実行計画を確認

```
terraform plan
```

反映しても問題ないか実行計画を確認

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.s3-bucket will be created
  + resource "aws_s3_bucket" "s3-bucket" {
      + acceleration_status         = (known after apply)
      + acl                         = (known after apply)
      + arn                         = (known after apply)
      + bucket                      = "localstack-test-bucket"
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

## 4. S3バケットにファイルをアップロードしてみる

### 4-1. aws-cli実行環境に入る

```
docker compose exec aws-cli sh
```

### 4-2. アップロードする

```
aws --endpoint-url=http://localstack:4566 s3 cp hello.html s3://localstack-test-bucket/
```

以下の出力があればアップロード成功

`upload: ./hello.html to s3://localstack-test-bucket/hello.html`

### 4-3. アップロードされているか確認

以下のURLをブラウザから確認

```
http://localhost:4566/localstack-test-bucket/hello.html
```

以下の画面が表示されればOK

![スクリーンショット 2025-06-23 9 48 32](https://github.com/user-attachments/assets/abd69ce1-321d-428c-9723-7cff469208eb)

## 5. S3バケットの変更を試してみよう

### 5-1. terraform実行環境に入る

```
docker compose exec terraform sh
```

### 5-2. `s3.tf` を編集して、バケットにタグを追加

```
resource "aws_s3_bucket" "s3-bucket" {
  bucket = "localstack-test-bucket"

  tags = {
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}
```

### 5-3. 実行計画を確認

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

### 5-4. AWSに反映

```
terraform apply
```

以下の出力があれば反映成功

`Apply complete! Resources: 0 added, 1 changed, 0 destroyed.`

### 5-5. AWS上で確認

aws-cli実行環境に入る

```
docker compose exec aws-cli sh
```

バケットに紐づくタグを取得する

```
aws --endpoint-url=http://localstack:4566 s3api get-bucket-tagging --bucket localstack-test-bucket
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

