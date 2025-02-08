# aws-dop-c02-notes

## AWS WAFのログ記録
- ログ記録を有効にし、WAF ログ記録の宛先として Amazon S3 バケット ARN を指定。
- バケット名は、任意のサフィックスで始まりaws-waf-logs-、任意のサフィックスで終わる必要があります。

! AWS WAF は、AWS によって管理される AWS Key Management Service キーの暗号化をサポートしていません。
! AWS では、Data Firehose を WAF ログ記録先として設定する場合、Kinesis ストリームをソースとして使用しないことを推奨しています。

## S3 アクセスパターン分析
- S3 バケットでサーバーアクセスのログ記録を有効にします。
- Amazon Athena を設定して、ログファイルを含む外部テーブルを作成します。SQL クエリを使用して、Athena からのアクセスパターンを分析します。

! Amazon Redshift Spectrum は、現在のユースケースに必要な単純なアクセスパターン分析にはコストがかかりすぎます。
! シンプルなアクセスパターン分析に AWS Lambda、Kinesis Data Firehose、および Amazon OpenSearch サービスを使用するため、これはコスト効率の高いソリューションではありません。

## セキュリティに違反したS3の自動修復
- AWS ConfigルールのAWS Config自動修復を設定しますs3-bucket-logging-enabled。修復アクションリストからAWS-ConfigureS3BucketLogging
- 修復アクションパラメータAutomationAssumeRoleは SSM によって引き継がれる必要があります。ユーザーは AWS Config で修復アクションを作成するときに、そのロールのパスロール権限を持っている必要があります。

! 今回のケースはSSMやLambdaは必要ない。AWSConfigの修復アクションリストに入っている

## CodeDeployからLambdaをデプロイしたがDBの変更が伝達していなかった
- BeforeAllowTrafficトラフィックが新しいバージョンの Lambda 関数に流れる前に、必要なデータベースの変更をテストして待機するフックを AppSpec ファイルに追加します。

! LambdaのDeployは BeforeAllowTraffic -> AllowTraffic -> AfterAllowTrafic
