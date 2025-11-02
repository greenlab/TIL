# Cloud Run メモ

## 簡単な流れ

### Cloud Shell を使用した場合

Google Cloud Platform 上のCloud Shell を使用した場合。

参考：https://codelabs.developers.google.com/codelabs/cloud-run-dev2prod?hl=ja#0

1. GCP上にプロジェクトを作成する。

2. 課金アカウントを紐づける。

3. Cloud Shell 上の事前設定

	1. 環境変数設定
		```
		export PROJ=$GOOGLE_CLOUD_PROJECT 
		export APP=hello 
		export PORT=8080
		export REGION="us-central1"
		export TAG="gcr.io/$PROJ/$APP"
		```
	2. API有効化
		```
		gcloud services enable cloudbuild.googleapis.com         \
		                       containerregistry.googleapis.com  \
		                       run.googleapis.com    
		```

4. Webアプリを作成する。

5. dockerによるコンテナ化を行う。

	1. Dockerfile を作成する。
	2. コンテナイメージを作成
		```
		gcloud builds submit --tag $TAG
		```
		
		$TAG は環境変数。
	3. コンテナを実行する。
		```
		docker run -p $PORT:$PORT -e PORT=$PORT $TAG	
		```

6. Cloud Run にデプロイする。

	1. Cloud Run にデプロイする。
		```
		gcloud run deploy "$APP"   \
		  --image "$TAG"           \
		  --platform "managed"     \
		  --region "$REGION"       \
		  --allow-unauthenticated
		```

7. Buildpack によるコンテナの自動作成

	Cloud Run は。Buildpacksというオープンソースによるコンテナ作成ツールをサポートしているとのこと。
	
	**Buildpacks** とは。 
	
	1. Procfile を作成する。
		```
		web: python3 main.py
		```
	2. Cloud Run にデプロイする。
		```
		gcloud beta run deploy "$APP"  \
		    --source .                 \
		    --platform "managed"       \
		    --region "$REGION"         \
		    --allow-unauthenticated
		```

8. アプリの負荷テスト

	hey というテストツールで行う。  
	※秋葉原のゲームセンターかと思ったが…。  
	
	cloud run に標準装備と書かれていたが、コマンド認識しなかったので、インストールから  
	go 言語が必要だが、cloud run は標準装備のもよう。
	
	1. インストール
		```
		go install github.com/rakyll/hey@latest
		```
	2. テスト
		```
		hey -q 1000 -c 200 -z 30s https://hello-...run.app
		```

9. クリーンナップ

	課金対象とならないよう、テストしたら削除。
	
	コンテナイメージ削除  
	```
	gcloud container images delete $TAG
	```
	
	Cloud Run削除
	```
	gcloud run services delete hello --platform managed --region $REGION --quiet
	```

