# 利用方法

1. oogiri-system/init.yml のパラメータ AllowedIAMRoleArn、ImageBucketName を書き換えます
    - AllowedIAMRoleArn：S3 バケットへのオブジェクトアップロードを許容する IAM エンティティの ARN
    - ImageBucketName：S3 バケット名
2. oogiri-system/init.yml を CloudFormation テンプレートとして新規スタックをデプロイします。
3. システムの稼働に必要な一式が作成されます
    - S3 バケット：Web システムの稼働用途の静的ウェブホスティングとして利用、大喜利で生成した画像のアップロード先として利用
      - S3 バケットポリシー：AllowedIAMRoleArn で指定したエンティティからの PutObject を許容するポリシー、静的ウェブホスティングとして稼働するためのバケットポリシーを設定
      - S3 イベント通知：画像のアップロードを検知し、後述の DynamoDB に画像のメタデータを格納する
    - DynamoDB：ImageMetadata として生成されます。S3 へのオブジェクトアップロードをトリガーにメタデータを格納します。
    - Lambda 関数：
      1. S3 イベント通知による DynamoDB へのメタデータ登録
      2. Web システムからの画像取得（一覧）
      3. Web システムからの画像取得（任意の単一画像）
      4. Web システムから送信されたデータによるいいね数の追加
    - API Gateway：上記 Lambda 関数のうち <2>、<3>、<4> を受け付けるためのエンドポイントとして生成

# 利用想定

`s3://<replace your s3 bucket name>/` に対して、オオギリストユーザー名、お題番号、をプレフィックスに持つ形式でお題の生成画像をアップロードしてもらう想定（例：`s3://<replace your s3 bucket name>/kazzpapa3/1/image.png` ）