# example-redash

## 前提条件
- Docker for Mac (Enable Kubernetes)
- kubectl
- kompose
- direnv [任意]

## オリジナルとの変更点
- docker-composeのバージョンを3.7に変更
- mailhogを追加
- Kubernetes対応

## セットアップ
### docker-compose
0. COMPOSE_PROJECT_NAMEの設定 (direnvがインストールされていれば自動で設定されている) [任意]
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

3. 起動
```
docker-compose up -d
```

4. Admin Userの登録

- http://localhost

5. メール送信のテスト
```
docker-compose run --rm server manage send_test_mail
```

6. メールの確認 (MailHogにメールが届いていたらOK)

- http://localhost:8025

7. 終了
```
docker-compose stop
```

### k8s
1. docker-compose.ymlからk8sのマニフェストファイルの雛形を作成
```
kompose convert -o k8s/overlays/local
```

2. ExternamNameを作成する (komposeがdocker-composeのexternal_linkの変換に対応していないため)
```
# k8s/overlays/local/externalname.yaml
apiVersion: v1
kind: Service
metadata:
  name: redash
spec:
  type: ExternalName
  externalName: server.<namespace>.svc.cluster.local.
```

3. ServiceのTypeをNodePortにする
```
perl -i.bak -pe 's/^spec:/spec:\n  type: NodePort/' k8s/overlays/local/*-service.yaml
rm -f k8s/overlays/local/*.bak    # 確認して問題なければバックアップを削除
```

4. applyする
```
kubectl apply -f k8s/overlays/local
```

5. 正常に起動していることを確認
```
kubectl get po,deploy,svc,ing
```

6. DBの初期化
```
kubectl exec -it server-<id> bin/docker-entrypoint create_db
```

7. Admin Userの登録

- http://localhost:<NginxのNodePort>

8. メール送信のテスト
```
kubectl exec -it server-<id> bin/docker-entrypoint manage send_test_mail
```

9. メールの確認 (MailHogにメールが届いていたらOK)

- http://localhost:<MailHogのNodePort>

10. 終了
```
kubectl delete -f k8s/overlays/local
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
