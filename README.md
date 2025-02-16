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

##  オンプレミスのデータを Amazon S3 バケットに移行する際の整合性検証
- コマンドのContent-MD5パラメータ内にMD5ダイジェストを指定しますPUT。Amazon S3の呼び出し戻りステータスを調べてエラーがないか確認します。
- 返されたレスポンスのETagを調べます。オブジェクトのETag値を、計算された、または以前に保存されたContent-MD5ダイジェストと比較します。

## インターネットにアクセスできないEC2インスタンスの集中アクセスと監視
- EC2 Image Builder を使用して、最新の AWS Systems Manager Agent バージョンを含むカスタム AMI を再構築します。Auto Scaling グループを設定して、AmazonSSMManagedInstanceCore ロールを EC2 インスタンスにアタッチします。
- Systems Manager Session Manager を使用して、集中管理された自動ログインを実現します。セッションの詳細を Amazon S3 にログ記録するように設定します。新しいファイルのアップロードに関する S3 イベント通知を設定し、Amazon Simple Notification Service (Amazon SNS) トピックを介してセキュリティ チームに通知します。

## Windows サーバーと Linux サーバーの大規模な混合フリートに対して AWS クラウドでのパッチ適用計画
- すべてのインスタンスにSystems Manager Agentを設定してパッチ適用を管理します。本番環境でパッチをテストし、適切な承認を得てメンテナンスウィンドウタスクとして展開します。
- AWS-RunPatchBaseline SSM ドキュメントを使用してパッチベースラインを適用する

! AWS-ApplyPatchBaseline SSM ドキュメントは、Windows インスタンスへのパッチ適用のみをサポート

## 異なる AWS リージョンへのアプリケーションのデプロイメントを自動化するための最適なソリューション
- リソースセクションで、アプリケーションインフラストラクチャを記述する CloudFormation テンプレートを作成します。管理者アカウントから CloudFormation スタックセットを使用して、アプリケーションを他のさまざまなリージョンにデプロイするスタックインスタンスを起動します。

## RTO を 3 時間、RPO を約 15 分。コスト効率の高い災害復旧 (DR) 戦略
- パイロット ライト DR 戦略を選択します。コア ワークロード インフラストラクチャのコピーを別の AWS リージョンにプロビジョニングします。
- 新しいリージョンに RDS リードレプリカを作成し、新しい環境がローカル RDS PostgreSQL DB インスタンスを指すように構成します。Amazon Route 53 ヘルスチェックを構成して、新しいリージョンへの DNS フェイルオーバーを自動的に開始します。災害が発生した場合に備えて、リードレプリカをプライマリ DB インスタンスに昇格します。

! RPO が 15 分の場合、データベースに増分バックアップを適用することはできません。
! アクティブ/アクティブ、ウォームスタンバイはコストが高い

## 組織内で後で作成される新しい AWS アカウントを自動的にオンボードする方法。
- Amazon CloudWatch のクロスアカウントの可観測性を使用して、セキュリティおよび運用アカウントを監視アカウントとして設定し、AWS Organizations を使用して組織の残りのメンバーアカウントとリンクします。

## CloudFormation スタックで作成されたリソースを削除せずに CloudFormation スタックの名前を変更するには
- スタックのすべてのリソースをデプロイする CloudFormation スタックを起動します。
- Retainこれらの各リソースの削除ポリシーに属性を追加します。元のスタックを削除します。別の名前で新しいスタックを作成し、元のスタックから保持されたリソースをインポートします。Retainスタックから属性を削除して、元のテンプレートに戻します。

! 「スナップショット」属性は、スナップショットを作成した後もリソースを削除します。

## ファイアウォール アプライアンスが重大度が「Critical」のイベントを生成した場合にセキュリティ チームにアラートを設定する
- ログイベントから「Critical」という用語に一致するようにログデータをフィルタリングして、Amazon CloudWatch にメトリクスフィルターを作成します。
- 検出結果のカスタムメトリクスを公開し、このカスタムメトリクスに CloudWatch アラームを設定して、Amazon Simple Notification Service (Amazon SNS) トピックに通知を公開します。セキュリティチームのメールアドレスを SNS トピックにサブスクライブします。

! ランジットゲートウェイフローログがIPトラフィックに関する情報で、今回のログデータ分析の目的には合わない。

## Auto Scaling グループ (ASG) が設定された Amazon EC2 Windows インスタンスがスケールインされたインスタンスを終了する前に、AMI を作成し、Amazon EC2 Windows インスタンスをそのドメインから削除したい。
- AWS Systems Manager 自動化ドキュメントを CloudWatch イベントターゲットとして追加します。自動化ドキュメントは Windows PowerShell スクリプトを実行してコンピュータをドメインから削除し、EC2 インスタンスの AMI を作成します。
- インスタンスをステータスに設定するライフサイクルフックを追加し、ステータスをTerminating:Wait監視するためのAmazon CloudWatchイベントを設定します。

## 管理者以外のユーザーが手動でアクセスした Amazon EC2 インスタンスを 1 時間以内に終了することを義務付けるセキュリティ ポリシーを導入
- CloudWatch Logs サブスクリプションを作成して、Amazon EC2 インスタンスのログインイベントデータを AWS Lambda 関数に配信します。
- ログインイベントを生成した EC2 インスタンスに廃止タグを追加するように Lambda 関数を設定します。Amazon EventBridge ルールをスケジュールして、1 時間ごとに別の Lambda 関数を呼び出し、廃止タグが付いたすべての EC2 インスタンスを終了します。

## ASG(スポット インスタンスとオンデマンド インスタンス)でスケールインアクティビティ中に、ASG は、すでに他のインスタンスよりも少ないインスタンスを持つアベイラビリティゾーン (AZ) のインスタンスを終了しました。　一定期間、ASG はグループの指定された最大容量を超えました。
- 混合インスタンスポリシーを持つAuto Scalingグループがスケールインすると、Amazon EC2 Auto Scalingはまず、2つのタイプ（スポットまたはオンデマンド）のどちらを終了するかを特定します。これにより、AZ間の不均衡が一時的に発生する可能性があります。
- Amazon EC2 Auto Scaling は、再バランス調整アクティビティ中に、グループの指定された最大容量を一時的に 10 パーセントのマージン (または 1 つのインスタンスのマージンのいずれか大きい方) 超過することがあります。

! 再バランス調整を行う際、Amazon EC2 Auto Scaling は古いインスタンスを終了する前に新しいインスタンスを起動する

## AWS Lambda 関数のレイテンシーを常に短縮するためにどのように設定すればよいか(DAXは設定済み)
- プロビジョニングされた同時実行を使用するように AWS Lambda 関数を設定します。プロビジョニングされた同時実行の値をそれぞれ 1 (最小) と 100 (最大) にして、Lambda 関数のアプリケーションの Auto Scaling を設定します。

## CodeDeploy Blue/Green デプロイメントを使用して新しいバージョンをデプロイしているときに、AllowTrafficライフサイクル イベント中にデプロイメントが失敗したが、デプロイログにエラーは見当たらなかった。
- この問題の原因は、アプリケーションロードバランサー（ALB）のヘルスチェックの設定が間違っていることです。

## a) サーバーレス アプリケーションは、次のトラフィック パターンで AWS Lambda 上に構築されています。アプリケーションのトラフィックは水曜日に増加し始め、木曜日も高いままで、金曜日に減少し始めます。b) 別の主力アプリケーションは Spot Fleet で実行されます。アプリケーションの負荷が変化しても、フリートの CPU 使用率は約 50% に維持される必要があります。
- Spot Fleet で実行されるアプリケーションの平均 CPU 使用率を 50% にすることを目標とするターゲット追跡自動スケーリングポリシーを作成します。AWS CLI、SDK、または CloudFormation を使用してターゲット追跡を作成および管理できます。
- スケジュールされたスケーリングのAuto Scalingポリシーを使用し、AWS Lambda関数をスケーラブルなターゲットとしてスケジュールされたアクションを作成します。要件に基づいて最小容量と最大容量を指定します。AWS CLI、SDK、またはCloudFormationを使用して、スケジュールされたスケーリングを構成できます。

! ターゲット追跡スケーリングポリシーでは、ターゲット追跡スケーリングポリシーで使用される CloudWatch アラームを作成、編集、または削除しません
! ステップスケーリングポリシーは、DynamoDB、Amazon Comprehend、Lambda、Amazon Keyspaces、Amazon MSK、ElastiCache、または Neptune ではサポートされていません。

## AWS アカウントで使用されるすべての EBS ボリュームのバックアップを強制するには
- AWS アカウントで AWS Config を設定します。AWS::EC2::Volumeカスタムタグが EBS ボリュームに適用されていない場合はコンプライアンス違反を返すリソースタイプのマネージドルールを使用します。
- カスタム AWS Systems Manager Automation ドキュメント (ランブック) を使用して、事前定義されたバックアップ頻度でカスタムタグをすべての非準拠 EBS ボリュームに適用する修復アクションを設定します。

## アカウント B の Lambda 関数を使用してアカウント A の Amazon EFS を引き続き使用するために支援
- VPC と EFS ファイルシステムにアクセスする権限を持つ Lambda 実行ロールを更新する
- アカウント A をアカウント B に接続するための VPC ピアリング接続を作成します。EFS ファイルシステムポリシーを更新して、アカウント B にアカウント A の EFS ファイルシステムのマウントと書き込みのアクセス権を付与します。

## 、Amazon API Gateway、AWS Lambda、Amazon DynamoDB サービスを使用してグローバル ユーザー ベースに高い信頼性と低レイテンシーを提供するクイック スタート ソリューション
- mazon Route 53 を設定して、ヨーロッパおよびアジア太平洋地域の API Gateway API を参照します。Route 53 でレイテンシーベースのルーティングとヘルスチェックを使用します。
- それぞれのリージョンの AWS Lambda 関数にリクエストを転送するように API を設定します。DynamoDB グローバル テーブルのデータを取得および更新するように Lambda 関数を設定します。

! 位置情報ルーティングを使用すると、コンテンツをローカライズし、Web サイトの一部またはすべてをユーザーの言語で表示できますが、レイテンシーを最小にしてコンテンツを提供するのには適していません。
! フェイルオーバールーティングは可能な限り低いレイテンシーでコンテンツを提供することが重要な要件である場合は適していません。
! AWS Global Accelerator を Amazon API Gateway と組み合わせて使用​​することで、インターネット向けの API を静的 IP アドレス経由でエンド ユーザーに提供できます。この設計は静的 IP セーフ リストの必要性に対応していますが、特定の要件には役立ちません。

## AWS アカウント間のデプロイメントのために、AWS CodePipeline を使用して、AWS アカウント (アカウント A) の AWS CloudFormation スタックを別の AWS アカウント (アカウント B) にデプロイする
- アカウントAで、アカウントAのCodePipelineサービスロールとアカウントBに使用権限を付与するカスタマー管理のAWS KMSキーを作成します。また、アカウントBにバケットへのアクセスを許可するバケットポリシーを持つAmazon Simple Storage Service (Amazon S3)バケットを作成します。
- アカウント B で、クロスアカウント IAM ロールを作成します。アカウント A で、AssumeRoleアカウント A の CodePipeline サービスロールに権限を追加して、アカウント B でクロスアカウントロールを引き受けられるようにします。
- アカウント B で、スタックによってデプロイされたサービスに必要な権限を含む CloudFormation スタックのサービスロールを作成します。アカウント A で、CodePipeline 構成を更新して、アカウント B に関連付けられたリソースを含めます。

## a) Cloudformation カスタムリソースのドリフトを検出する方法 b) スタックのドリフトステータスが CloudFormation コンソールで IN_SYNC と表示され、以下がドリフト検出エラー
- AWS Configルールは、CloudFormation APIのアクションの可用性に依存しますDetectStackDrift。AWS Configは、スロットリングが発生するとルールをNON_COMPLIANTにデフォルト設定します
- AWS CloudFormation はカスタムリソースのドリフト検出をサポートしていません

## AWS Glue ジョブの再試行が失敗したときに、Amazon Simple Notification Service (Amazon SNS) 通知を介して通知を受け取りたい
- AWS Glue の Amazon EventBridge イベントを設定します。AWS Lambda 関数を EventBridge のターゲットとして定義します。
- Lambda 関数には、イベントを処理し、AWS Glue ジョブ再試行失敗イベントをフィルタリングするロジックがあります。このようなイベントが見つかった場合は、Amazon Simple Notification Service (Amazon SNS) 通知にメッセージを公開します。

! Amazon EventBridge は Lambda 関数なしで直接使用することはできません。

## ダウンタイムなしのBlue/Greenデプロイと2時間のウインドウ、ウインド終了度の自動終了のソリューション
- AWS CodeDeploy を、Blue/Green デプロイメント構成に設定されたデプロイメントタイプで使用します。
- 2 時間後に元のフリートを終了するには、Blue/Green デプロイメントのデプロイメント設定を変更します。Original instances値を に設定しTerminate the original instances in the deployment group、待機期間を 2 時間選択します。

## AWS Access Key ID、、Secret Access Keyおよびデータベースパスワードを参照する環境変数の値がハードコードされていることに気付きました。さらに、ビルドフェーズ中に 1 回限りの構成変更を実行するために、ファイルにはssh、scpAmazon S3 に保存されている SSH 秘密キーを使用して EC2 インスタンスとの間で送受信されるコマンドが含まれています。
- AWS Systems Manager コマンドを利用して EC2 インスタンスを管理runします。ssh scp
- CodeBuild プロジェクトロールに必要な権限ポリシーを設定し、buildspec.yaml ファイルから AWS 認証情報を参照する環境変数を削除します。
- データベースのパスワードを、AWS Systems Manager パラメータストアに SecureString 値として保存し、buildspec 環境で参照します。また、buildspec.yaml ファイルから、データベースのパスワードのハードコードされた値を参照する環境変数を削除します。

! AWS Secrets Manager には SecureString 値はありません。

## CloudFormation スタックは進行中のステータス (CREATE_IN_PROGRESS) から完了ステータス (CREATE_COMPLETE) に移行していません。
- AWS Lambda 関数を設定して、カスタムリソース作成の応答 (成功または失敗) を、事前に署名された Amazon Simple Storage Service URL に送信します。

! カスタムリソースプロバイダーは JSON 形式のファイルで応答し、署名済み S3 URL にアップロードします。この URL が指定されていない場合、呼び出しテンプレートは Lambda 関数のステータスの更新を取得せず、進行中の状態のままになります。

##  (ALB) と Amazon API Gateway API に対して AWS Web アプリケーションファイアウォール (AWS WAF) Web ACL を構成を自動化する
- 組織内の AWS アカウントの 1 つを AWS Organizations の Firewall Manager の管理者として指定します。AWS Firewall Manager ポリシーを作成して、新しく作成された ALB と API Gateway API に AWS WAF ウェブ ACL をアタッチします。

! AWS Config はリソースのステータスを追跡できますが、セキュリティインフラストラクチャを集中管理するには AWS Firewall Manager が必要です。

## CloudFront、API Gateway、Lambda 関数で構成されるサーバーレス アプリケーション スタックがあります。この会社は、Lambda 関数の新しいバージョンを作成し、AWS CLI スクリプトを実行してデプロイする現在のデプロイ プロセスを改善
- サーバーレスアプリケーションモデル (SAM) を使用し、SAM の組み込みトラフィックシフト機能を活用して、CodeDeploy 経由で新しい Lambda バージョンをデプロイし、トラフィック前およびトラフィック後のテスト機能を使用してコードを検証します。CloudWatch アラームがトリガーされた場合はロールバックします。

! CodeDeploy を使用すると、新しい Lambda バージョンを公開するがトラフィックを送信しないデプロイ プロセスを作成できます。次に、PreTraffic テストを実行して、新しい関数が期待どおりに動作することを確認します。テストが成功すると、CodeDeploy はトラフィックを徐々に新しいバージョンの Lambda 関数に移行します。
! CloudFormation 変更セットを使用するとスタックを実際にデプロイするまで潜在的な障害について知ることができない

## AWS CloudFormation スタックの更新プロセス中スタックが UPDATE_ROLLBACK_FAILED 状態に、完了させるには？
- スタックの正しい状態に一致するようにリソースを手動で修正する
- AWS CloudFormationからContinueUpdateRollbackコマンドを実行する

! ドリフト検出は、予想されるテンプレート設定から外れたリソースを識別するのに役立ちますが、UPDATE_ROLLBACK_FAILED 状態の解決には直接関係しません。
