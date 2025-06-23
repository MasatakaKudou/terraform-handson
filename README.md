# terraform-handson

ローカルでterraformを試そう

## 0. 事前準備

`docker compose` or `docker-compose` を使える

## 1. ハンズオン環境の準備

1-1. クローンする

`git clone git@github.com:MasatakaKudou/terraform-handson.git`

1-2. プロジェクトに移動しコンテナ起動

`docker compose up -d` or `docker-compose up -d`

以降は `docker compose` をベースに説明していく

1-3. 期待しているコンテナが立ち上がっているか確認

`docker compose ps`

以下コンテナが立ち上がる想定

| TH | TH |
| ---- | ---- |
| aws-cli | aws-cli実行環境 |
| localstack | 擬似AWS環境 |
| terraform | terraform実行環境 |

`STATUS` が全てUpになってればOK

## 
