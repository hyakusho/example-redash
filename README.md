# example-redash

## 前提条件
- Docker for Mac (Enable Kubernetes)
- direnv (任意)

## オリジナルとの変更点
- docker-composeのバージョンを3.7に変更
- mailhogを追加
- Kubernetes対応

## セットアップ
0. COMPOSE_PROJECT_NAMEの設定 (任意、direnvがインストールされていれば自動で設定されている)
```
export COMPOSE_PROJECT_NAME=readsh
```

1. .envファイルの作成 (必要に応じて編集)
```
cp .env.default .env
```

2. DBの作成
```
docker-compose run --rm server create_db
```

3. メール送信のテスト
```
docker-compose run --rm server manage send_test_mail
```

4. 起動
```
docker-compose up -d
```

5. 終了
```
docker-compose stop
```

## Tips
### Redashのバージョン一覧
```
curl -s 'https://version.redash.io/api/releases?channel=stable' | jq -r .
```

### Redashの最新バージョンのイメージを取得
```
curl -s 'https://version.redash.io/api/releases?channel=stable' | jq -r '.[0].docker_image'
```

### オリジナルのdocker-compose.ymlの取得
```
curl -sLO 'https://raw.githubusercontent.com/getredash/setup/master/data/docker-compose.yml'
```
