# terraform-handson
ローカルでterraformを試そう

## 1. ハンズオン環境の準備

クローンする

`git clone git@github.com:MasatakaKudou/terraform-handson.git`

プロジェクトに移動しコンテナ起動

`docker compose up -d` or `docker-compose up -d`

以下コンテナが立ち上がる

| TH | TH |
| ---- | ---- |
| localstack | 擬似AWS環境 |
| terraform | terraform実行環境 |
| aws-cli | aws-cli実行環境 |
