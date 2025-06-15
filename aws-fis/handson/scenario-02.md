# 障害は忘れた頃にやってくる「AWS Fault Injection Service」で障害を食らってみよう / AWS FIS の設定と実験１

JAWS-UG 神戸 #7 で実施予定の AWS Fault Injection Service ハンズオン用シナリオです。

## AWS FIS の設定

以降の操作は CloudShell で実行することを想定しています。

### AWS FIS 用の信頼関係ドキュメントの作成

信頼するエンティティとして AWS FIS（fis.amazonaws.com）を持つ信頼関係ドキュメントを作成します。

```bash
cat << EOF > fis-trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "fis.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
```

### AWS FIS 用の IAM ロールの作成とポリシーのアタッチ

後述する実験ドキュメントで指定し、実験のためのリソース操作を行う IAM ロールを作成します。

そして実験として EC2 に対する操作、RDS に対する操作を行いたいため、AWS で用意している `AWSFaultInjectionSimulatorEC2Access` ポリシーと `AWSFaultInjectionSimulatorRDSAccess` ポリシーをアタッチします。

```bash
aws iam create-role \
  --role-name FISServiceRole \
  --assume-role-policy-document file://fis-trust-policy.json

aws iam attach-role-policy \
  --role-name FISServiceRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSFaultInjectionSimulatorEC2Access

aws iam attach-role-policy \
  --role-name FISServiceRole \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSFaultInjectionSimulatorRDSAccess
```

### 実験テンプレートの作成1（RDSに対する操作の実験）

AWS マネジメントコンソールで CloudShell を起動し、以下のコマンドを実行します。

このテンプレートでは、現在稼働している DB インスタンスのアベイラビリティゾーンと同一の AZ 内のリソース全てを対象とした DB インスタンスの再起動を実行します。  

> [!WARNING]  
> コマンド自体は「事前準備1」の CloudFormation テンプレートをそのまま利用されていることを想定し、DB 識別子が「aurora-serverless-cluster-instance」であることを前提として組み立てています。  
> DB 識別子名を変更している場合は適宜読み替えてください。

```bash
DB_INSTANCE_AZ=$(aws rds describe-db-instances --query "DBInstances[?DBInstanceIdentifier=='aurora-serverless-cluster-instance'].AvailabilityZone" --output text)
ROLE_ARN_FOR_FIS=$(aws iam get-role --role-name FISServiceRole --query 'Role.Arn' --output text)
cat << EOF > fis-experiment-template-for-rds.json
{
    "description": "Reboot RDS",
    "targets": {
        "DBInstances-Target-1": {
            "resourceType": "aws:rds:db",
            "selectionMode": "ALL",
            "parameters": {
                "availabilityZoneIdentifiers": "${DB_INSTANCE_AZ}"
            }
        }
    },
    "actions": {
        "RDS": {
            "actionId": "aws:rds:reboot-db-instances",
            "parameters": {
                "forceFailover": "false"
            },
            "targets": {
                "DBInstances": "DBInstances-Target-1"
            }
        }
    },
    "stopConditions": [
        {
            "source": "none"
        }
    ],
    "roleArn": "arn:aws:iam::720791945764:role/FISServiceRole",
    "tags": {},
    "experimentOptions": {
        "accountTargeting": "single-account",
        "emptyTargetResolutionMode": "fail"
    }
}
EOF

aws fis create-experiment-template \
    --cli-input-json file://fis-experiment-template-for-rds.json
```

#### 補足


<details><summary>マネジメントコンソールで実施する場合は以下となります。</summary>

あとでかく

</details>


### 実験の実施





### 実験

### 実験テンプレートを作成する

```bash
ROLE_ARN_FOR_FIS=$(aws iam get-role --role-name FISServiceRole --query 'Role.Arn' --output text)
cat << EOF > fis-experiment-template.json
{
        "description": "1 つ以上のインスタンスを 2 分間停止します",
        "targets": {
                "TaggedInstances": {
                        "resourceType": "aws:ec2:instance",
                        "resourceTags": {
                                "Name": "fis-EC2-Instance"
                        },
                        "filters": [
                                {
                                        "path": "State.Name",
                                        "values": [
                                                "running"
                                        ]
                                }
                        ],
                        "selectionMode": "COUNT(1)"
                }
        },
        "actions": {
                "StopAction": {
                        "actionId": "aws:ec2:stop-instances",
                        "description": "特定のタグを持つインスタンスを対象にする",
                        "parameters": {
                                "startInstancesAfterDuration": "PT2M"
                        },
                        "targets": {
                                "Instances": "TaggedInstances"
                        }
                }
        },
        "stopConditions": [
                {
                        "source": "none"
                }
        ],
        "roleArn": "${ROLE_ARN_FOR_FIS}",
        "tags": {},
        "experimentOptions": {
                "accountTargeting": "single-account",
                "emptyTargetResolutionMode": "fail"
        }
}
EOF

aws fis create-experiment-template \
    --cli-input-json file://fis-experiment-template.json
```

## 環境の作り替え


### EC2 インスタンスへの接続

AWS マネジメントコンソールから CloudShell を起動し、以下のコマンドを実行します

```bash
KEY_NAME=$(aws ssm describe-parameters --query "Parameters[?contains(Name, 'keypair')].Name" --output text)
aws ssm get-parameters --names "${KEY_NAME}" --with-decryption --query "Parameters[].Value" --output text > key.pem
chmod 400 key.pem
PUBLIC_IP=$(aws ec2 describe-instances   --filters "Name=instance-state-name,Values=running"   --query 'Reservations[].Instances[?contains(Tags[?Key==`Name`].Value | [0], `-EC2-Instance`)][].NetworkInterfaces[].Association.PublicIp' --output text)
ssh -i key.pem ec2-user@"${PUBLIC_IP}cd /var/www/html/"
```

### WordPres 設定の変更

```bash
cd /var/www/html/
GLOBAL_IP=$(curl inet-ip.info)
wp option update home "http://${GLOBAL_IP}"
wp option update siteurl "http://${GLOBAL_IP}"
```





### CloudWatch ダッシュボードの作成

```bash
aws cloudwatch put-dashboard \
    --dashboard-name FIS \
    --dashboard-body '{"widgets":[{"height":6,"width":6,"y":0,"x":0,"type":"metric","properties":{"view":"timeSeries","stacked":false,"metrics":[["Namespace","CPUUtilization","Environment","Prod","Type","App"]],"region":"ap-notheast-1"}}]}'
```





aws iam attach-role-policy \
  --role-name FISServiceRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess

aws iam attach-role-policy \
  --role-name FISServiceRole \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchFullAccessV2
