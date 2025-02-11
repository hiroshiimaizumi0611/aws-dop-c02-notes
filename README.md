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

## Auroraクラスターのメンテナンスで中断を最小限におさえるには
- リーダーインスタンスを追加。
- 書き込み操作に Aurora クラスターエンドポイントを使用するようにアプリケーション構成を更新します。読み取り操作用に Aurora クラスターリーダーエンドポイントを更新します。

! クラスター構成にしたらマルチAZに変更はできない。

## EC2とAWｓアカウントのAPIログの両方をクエリする方法
- AWS CloudTrail を設定、API ログを CloudWatch Logs に配信。
- Amazon CloudWatch Agent を活用して、EC2 インスタンスから Amazon CloudWatch Logs にログを配信。CloudWatch Logs Insights を使用して、両方のログ セットをクエリする。

## EC2 インスタンスのウェアの脆弱性と意図しないネットワーク露出をプロアクティブに検出するセキュリティ ソリューション
- Amazon Inspector を設定して、EC2 インスタンスの脆弱性を検出します。
- Amazon CloudWatch エージェントをインストールして、システムログをキャプチャし、Amazon CloudWatch Logs 経由で記録します。
- ログイベントを CloudWatch Logs に送信するようにトレイルを設定します。

! SM エージェントは、Amazon EC2 インスタンスの脆弱性をスキャンまたは検出しません。

## ポリシーを更新せずにすべての新しいリソースにアクセス制御を適用する方法
- タグを使用して属性ベースのアクセス制御を設定。
- IAMロールの機能に基づいてタグを作成します。新しいリソースが作成されるたびに、これらのタグをリソースに追加して、作成されたリソースにすぐにアクセスできるようにします。

## ALBの背後のEC2をブルー/グリーンデプロイをしたい

- グリーン環境の Auto Scaling グループのローリング再起動を開始して、グリーン環境の EC2 インスタンスに新しいソフトウェアをデプロイします
- ローリング再起動が完了したら、AWS CLI コマンドを利用して ALB を更新し、トラフィックをグリーン環境のターゲットグループに誘導します。

##  AWS アカウント内の制限のないセキュリティ グループに関するカスタマイズされた通知をリアルタイムで生成する自動監視ソリューションを作成

- 制限付き SSH ルールの AWS Config 評価結果が NON_COMPLIANT に一致する Amazon EventBridge ルールを設定。
- EventBridge ルールの入力トランスフォーマーを作成します。SNS トピックに通知を発行するように EventBridge ルールを設定。

! COMPLIANT, NON_COMPLIANT, ERROR, NOT_APPLICABLE

## EC2へのログイン発生時に通知
- すべてのログを Amazon CloudWatch Logs にプッシュするように設定
- 各 Amazon EC2 インスタンスに Amazon CloudWatch エージェントを設定します。
- ユーザーのログインを検出するための CloudWatch メトリックフィルターを作成します。ログインが検出されると、Amazon SNS を使用してセキュリティチームに通知します。

! CloudTrailはAPI呼び出しから5分以内にログ送信をするのでリアルタイム性に欠ける

## AWS リソースの構成を継続的に評価、監査、監視するためのビジネス要件に対処するための戦略
- AWS Config ルールを活用して、AWS リソースへの変更を監査し、選択した頻度でルールの評価を実行して設定のコンプライアンスを監視します
- 証跡を有効にして CloudTrail イベントを設定し、KMS キーを使用してこれらのアクティビティを CloudWatch Logs に記録することで、すべての AWS アカウントの管理アクティビティを確認および監視します。

## Amazon S3 デプロイアクションを備えた AWS CodePipeline を使用して、開発アカウントの S3 バケットから本番アカウントの S3 バケットに成果物をデプロイする
- 本番アカウントでクロスアカウントロールを設定します。開発アカウントのCodePipelineサービスロールにポリシーをアタッチして、作成したクロスアカウントロールを引き受けられるようにします。
- 開発アカウントで CodePipeline で使用する AWS KMS キーを作成します。また、開発アカウントの入力バケットでは、CodePipeline で動作するためにバージョン管理が有効になっている必要があります。
