# 障害は忘れた頃にやってくる「AWS Fault Injection Service」で障害を食らってみよう

JAWS-UG 神戸 #7 で実施予定の AWS Fault Injection Service ハンズオン用シナリオです。

## 前提

- 可能な限り発行したばかりの AWS アカウントの利用が望ましいです
  - VPC を一つ作成しますので、既存環境がある場合 Quotas に引っかかる可能性があります
- 東京リージョンでの実行を想定しています
- AWS CLI での操作は CloudShell で実施することを想定しています
- 2025年6月15日現在の AWS API 仕様に基づいて構築していますので、将来の AWS の変更によって挙動が変わる可能性があります

## 事前準備

### 事前準備1：CloudFormation テンプレートの実行

AWS FIS によって引き起こされる事象を喰らう環境として、粗末なウェブサーバを１台だけ持つインフラ環境を構築します。

環境が構築されるまで 15 分程度時間を要します。



#### マネジメントコンソールでの操作

1. [vpc-ec2-aurora-serverless-template-updated.yaml](./vpc-ec2-aurora-serverless-template-updated.yaml) をダウンロードしておきます
2. AWS マネジメントコンソールより CloudFormation へ遷移します。  
    なお、東京リージョン（ap-northeast-1）で実行されることを期待するテンプレートとなるため、リージョンが東京リージョン（ap-northeast-1）であることを合わせて確認します
3. 「スタックの作成」から「テンプレートの指定」を「テンプレートファイルのアップロード」とした上で ＜1＞ でダウンロードしたテンプレートをアップロードします
4. スタック名を適宜設定します。  
    なおパラメータの値はデフォルト値のままで良いですが、セキュリティ的に気になる点があれば、ご自身の判断で変更しても構いません。
5. 「次へ」ボタンをクリックします。

> [!WARNING]  
> この際、ブラウザによっては「パスワードを変更してください」などのパスワード マネージャーからの警告が出る可能性があります。  
> これは CloudFormation のパラメータ値として設定している Aurora MySQL の認証情報としてセルフマネージドのマスターパスワードに安直な文字列を使用していることに依存します。

#### AWS CLI での操作

```bash
wget https://raw.githubusercontent.com/kazzpapa3/jawsug-kobe/refs/heads/main/aws-fis/init.yaml
aws cloudformation create-stack \
  --stack-name init \
  --template-body file://init.yaml \
  --capabilities CAPABILITY_IAM
```

### 事前準備2：WordPress の設定

1. CloudFormation の「出力」タブを確認しながら、`EC2PublicIP` の値の IP アドレスに ウェブブラウザでアクセスする
2. 「さあ、始めましょう！」ボタンが表示されていることを確認し、ボタンをクリックする![setting2-1](./images/setting2-1.png)
3. データベース名、ユーザー名、パスワードは CloudFormation テンプレートで定義してあり、デフォルトから変更していなければ以下の通りとなります
    -  データベース名：`mydb`
    - ユーザー名：`admin`
    - パスワード：`Password123!`
4. データベースのホスト名は CloudFormation の「出力」タブにある `AuroraClusterEndpoint` の値を入力します
5. 「テーブル接頭辞」はデフォルトの「wp_」のままとし「送信」ボタンをクリックします![setting2-2](./images/setting2-2.png)
6. 次画面で「この部分のインストールは無事完了しました。WordPress は現在データベースと通信できる状態にあります。準備ができているなら…」と表示されていることを確認し「インストール実行」ボタンをクリックします![setting2-3](./images/setting2-3.png)
7. 「ようこそ」画面で「サイトのタイトル」「ユーザー名」「メールアドレス」を適宜入力し、「パスワード」として表示されているものをコピー（あるいは変更した上でコピー）し「WordPress をインストール」ボタンをクリックします![setting2-4](./images/setting2-4.png)
8. 「成功しました！」画面へ正しく遷移したことを確認します![setting2-5](./images/setting2-5.png)
9. ＜1＞ で確認した `EC2PublicIP` の値の IP アドレスに ウェブブラウザでアクセスすると「Hello world!」という記事だけが表示されている WordPress サイトが表示されることを確認しておきます

---

[AWS FIS の設定と実験１](./scenario-02.md) へ