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

## 企業は、新しく作成されたすべてのアカウントに必須の IAM ロールと CloudTrail 構成を追加し、アカウントが組織を離れたときに手動介入なしでリソース/構成を削除できる自動化ソリューションを探しています。
- AWS Organizations の管理アカウントから、AWS CloudFormation スタックセットを作成して AWS Config を有効にし、集中管理された AWS Identity and Access Management (IAM) ロールをデプロイします。AWS Organizations を通じてアカウントが作成されたときにスタックセットが自動的にデプロイされるように構成します。

! CloudFormation StackSets では、新しい AWS アカウントが組織に参加または組織を離れたときに、CloudFormation スタックを自動的に作成または削除するオプションが提供されます。

## 100 MB のオブジェクトを Amazon S3 バケットにシングルパート直接アップロードとしてアップロードしました。オブジェクトのチェックサムを確認したところ、チェックサムはオブジェクト全体のチェックサムではありませんでした。
- 個々の部分のチェックサム値に基づいて計算されたオブジェクトの新しいチェックサム値が作成されました。この動作は予想どおりです。

! オブジェクトのサイズが 16 MB を超えると、Amazon S3 はマルチパートアップロードを使用します。この場合、チェックサムはオブジェクト全体の直接のチェックサムではなく、個々の部分のチェックサム値に基づいて計算されます。

## CodeBuild 環境からクロス アカウント EKS クラスターへの接続を確立しようとすると、権限のないエラーが発生
- 集中型 DevOps アカウントのアプリケーション アカウントのデプロイメント IAM ロールに信頼関係を確立し、sts:AssumeRole アクションを許可します。
- また、アプリケーション アカウントのデプロイメント IAM ロールに EKS クラスターへの必要なアクセスを許可します。さらに、ロールを適切なシステム権限にマッピングするように EKS クラスターの aws-auth ConfigMap を構成します。

! AssumeRoleWithSAML は、SAML ベースの ID プロバイダーが関与するフェデレーション シナリオに使用されます

## 会社のセキュリティ ポリシーを遵守しながらアカウント B と AMI を共有する
- アカウント A で、暗号化されていないバージョンから暗号化された AMI を作成し、コピー アクションで KMS キーを指定します。キー ポリシーを変更して、アカウント B に権限付与を作成する権限を付与します。暗号化された AMI をアカウント B と共有します。
- アカウントBで、Auto Scalingグループにアタッチされたサービスリンクロールに権限を委任するKMS権限付与を作成します。

## AWS CodePipeline パイプラインを使用して新しい API が展開されるたびに、常に最新の SDK がクライアントに提供されるようにする自動化ソリューションを実現
- API デプロイ段階の直後に実行される CodePipeline アクションを設定します。このアクションを設定して AWS Lambda 関数を呼び出します。
- Lambda 関数は API Gateway から SDK をダウンロードし、S3 バケットにアップロードして、SDK パスの CloudFront 無効化を作成します。

## DevOps エンジニアが最小限の権限でスタックを展開できるようにするソリューション
- 必要な権限を持つAWS CloudFormationサービスロールを作成し、このサービスロールをスタックに関連付けます。開発者にiam:PassRoleロールをサービスに渡す権限を付与します。スタックのデプロイ中に、この新しく作成されたサービスロールを使用します。

## 最も適切な EC2 起動モードを使用していることを確認し、その決定に対する違反を強調表示するコンプライアンスダッシュボードを作成したい
- 専用ホストで EC2 インスタンスを起動し、アプリケーションのタグを作成します。
- アプリケーションタグをチェックし、インスタンスが正しい起動モードで起動されるようにする Lambda 関数によってサポートされる AWS Config カスタムルールをデプロイします。

! 課金目的で CPU ソケットにアクセスするには、EC2 専用ホストを使用する必要があります。
! リザーブドインスタンスは、EC2 の年間使用コストを節約するために用意されています。リザーブドインスタンス (RI) は、オンデマンド価格と比較して大幅な割引 (最大 72%) を提供し、特定のアベイラビリティーゾーンで使用する場合に容量を予約します。

##  Elastic Beanstalk を使用して、DNS 名が変更されず、新しいリソースが作成されないことを確認する必要があります
- 一度に20%ずつローリングアップデートを使用する

! B/Gだと新しリソースがデプロイされるからダメ？

## CodeCommit を採用しており、CICD システムでプル リクエストを自動的に作成し、合格/不合格のステータスを示すテスト バッジを提供する
- ソースリポジトリのプルリクエストの作成と更新に反応する CloudWatch イベントルールを作成します。そのルールのターゲットは CodeBuild である必要があります。
- CodeBuild ビルドの成功または失敗イベントを監視する 2 番目の CloudWatch イベントルールを作成し、ターゲットとしてプルリクエストをビルド結果で更新する Lambda 関数を呼び出します。

## 6 つのインスタンス、3 つのアベイラビリティ ゾーン (AZ) に分散された Apache Kafka クラスターを展開。Apache Kafka はステートフル サービスでありEBS ボリュームに保存する必要があります。各インスタンスには自動修復機能が必要であり、EBS ボリュームを接続する必要があります。
- 小/最大容量が 1 の ASG と EBS ボリュームを持つ CloudFormation テンプレートを作成します。ASG と EBS ボリュームにタグを付けます。
- 起動時に EBS ボリュームを取得するユーザー データ スクリプトを作成します。マスター CloudFormation テンプレートを使用し、ネストされたテンプレートを 6 回参照します。

## CloudFormation テンプレートをデプロイしようとしていますが、チームは というエラーを受けていますInsufficientCapabilitiesException
- Deploy CloudFormationステージアクションのCodePipeline構成でIAM機能を有効にする

! 指定されたユースケースでは、InsufficientCapabilitiesExceptionCloudFormation スタックが IAM ロールを作成しようとしているが、指定された機能がないことを意味します。

## トラブルシューティングのために、不確定な期間にわたって 1 つのインスタンスを分離できる必要があります
- インスタンスを起動後すぐにスタンバイに設定する

! InService 状態のインスタンスを Standby 状態にして、インスタンスを更新またはトラブルシューティングしてから、インスタンスをサービスに戻すことができます

## Multi-AZ を使用してセットアップされている MySQL データベースのメジャー バージョンをアップグレードしたいと考えています。ダウンタイムを可能な限り最小限に抑える必要がある
- CloudFormation テンプレートで RDS リードレプリカを作成しSourceDBInstanceIdentifier、作成されるのを待ちます。その後、RDS リードレプリカをEngineVersion次のメジャーバージョンにアップグレードします。次に、リードレプリカを昇格して、新しいマスターデータベースとして使用します。

! リードレプリカを使用したローリングアップグレードを使用すると、アップグレード時のダウンタイムを最小限に抑えることができます
! DBEngineVersionというのはひっかけ？

## DynamoDB テーブルに大量のデータがある場合、Lambda 関数でスロットリングの問題が発生
- ストリームから読み取り、ペイロードをSNSに渡す新しいLambda関数を作成します。他の3つのLambda関数と今後のLambda関数はSNSトピックから直接読み取ります。

## トラフィックの処理中に EC2 インスタンスの最大 CPU 使用率が異常に高くなった場合に、デプロイを段階的に実行し、自動的にロールバックすることを望んでいます。
- EC2 インスタンスの最大 CPU 使用率の CloudWatch メトリックを作成します。そのメトリックに基づいて CloudWatch アラームを作成します。CodeDeploy でロールバックを有効にし、CloudWatch アラームと統合したデプロイメントを作成します。

## CodeCommit リポジトリ内のコードにプッシュされるたびに、サードパーティのコード脆弱性分析ツールを開始したい
- CodeCommitリポジトリにプッシュに反応するCloudWatchイベントルールを作成します。ターゲットとして、CodeCommitからコードをリクエストし、それを圧縮してサードパーティAPIに送信するAWS Lambda関数を選択します。

## Jenkins セットアップが高可用性、耐障害性、およびビルド実行の弾力性を備えていることを確認する必要があります。
- 複数のAZにまたがるマルチマスター設定としてJenkinsをデプロイします。JenkinsのCodeBuildプラグインを有効にして、ビルドがCodeBuildビルドとして起動されるようにします。

! AWS クラウドでは、Jenkins のような Web アクセス可能なアプリケーションは通常、インスタンスを複数の AZ に分散し、Elastic Load Balancing (ELB) ロードバランサーを前面に配置することで、高可用性と耐障害性を実現するように設計されています。

## AWS Systems Manager を利用してオンプレミスのインスタンスと EC2 インスタンスを管理したいと考えています。フリートのサイズを効果的に管理したい
- インスタンスが SSM サービスで操作を呼び出せるように、IAM サービス ロールを作成しますAssumeRole。オンプレミス サーバーのアクティベーション コードとアクティベーション ID を生成します。
- これらの認証情報を使用してオンプレミス サーバーを登録します。SSM コンソールにプレフィックス「mi-」が付きます。

## Python ランタイムと OS パッチを含む基本 AMI があることを確認したいと考えています。その AMI は、アカウント内のすべてのリージョンからスケーラブルな方法でプログラム的に参照できるようにする必要 
- AMI ID を 1 つのリージョンの SSM パラメータ ストアに保存し、他のすべてのリージョンに AMI をコピーし、対応する AMI ID を SSM に保存する Lambda 関数を用意します。同じパラメータ ストア名を使用して、リージョン間で再利用できるようにします。
- 繰り返し可能な方法で AMI を作成するための SSM 自動化ドキュメントを作成します。

! AMI ID はリージョン スコープであるため、AMI はリージョン間でコピーする必要があり、各 SSM パラメータ ストアには異なる AMI ID 値が含まれます。ただし、すべてのリージョンで同じ SSM パラメータ ストア キーを使用できます。

## EBS バックアップ ワークフローを作成。そのワークフローでは、EBS ボリュームを取得し、そこからスナップショットを作成する必要があります。作成されたら、別のリージョンにコピーする。災害のために他のリージョンが使用できない場合は、そのバックアップを 3 番目のリージョンにコピーする。最終結果は、電子メール アドレスに通知
- AWS Step Function を作成します。各ステップを Lambda 関数として実装し、条件付きケースに対処するためにステップ間に失敗ロジックを追加します。

## ハイブリッド フリートのパッチ適用を自動化し、毎週社内のパッチ リポジトリを通じてパッチを配布したい
- SSMを使用して、カスタムパッチベースラインを実装します。メンテナンスウィンドウを定義し、実行コマンドRunPatchBaselineを含めます。メンテナンスウィンドウを毎週繰り返してスケジュールします。

! EC2 インスタンスに構成ファイルを書き込むことはできません

## タスクが完了するまでトラフィックが新しい Lambda 関数に提供されないようにするには
- ファイルにappspec.yml、BeforeAllowTrafficステップ関数の実行の完了を確認するフックを含めます。

## Auto-Scaling グループ (ASG) を介して主力アプリケーションを実行。グループの現在の平均 CPU 使用率は、オフピーク時には 15% ピーク時には 45% 、これらのピーク時間は営業時間中に予測どおりに発生します。この企業は、この要件に対応するソリューションを構築したい
- CPU 使用率を 75% を目標として追跡するスケーリング ポリシーを作成します。ピーク時に最小インスタンス数を 6 に増やすスケジュール アクションと、オフピーク時に最小インスタンス数を 3 に減らす 2 番目のスケジュール アクションを作成します。

## すべてのインスタンスにデプロイするための手動承認手順を実行する前に、メトリックを収集できるように、トラフィックをいくつかのインスタンスにデプロイする必要があります。
- CodeDeploy で、4 つのデプロイメント グループを作成します。1 つは開発用、1 つはステージング用、1 つは本番環境のカナリア テスト インスタンス用、もう 1 つは本番環境インスタンス全体用です。
- 1 つの CodePipeline を作成し、これらのステージを連結して、カナリア インスタンスへのデプロイメント後に手動承認手順を導入します。

! カナリアとは、本番システムに新機能をリリースする際、一部の限られたユーザーが被験体（カナリア）となるよう局所的に新機能を有効化していくことで、リリースに問題があった際の影響範囲を小さくとどめる手法です。

## CodeCommit と CodeBuild で構成される CodePipeline パイプラインを作成し、アプリケーションは Elastic Beanstalk にデプロイされています。チームは、CodePipeline を通じて Beanstalk のブルー/グリーン デプロイを有効にしたい
- CodePipeline を新しい Beanstalk 環境にデプロイします。そのステージアクションの後に、AWS Lambda を使用してカスタムジョブを呼び出す別のステージアクションを作成します。これにより、環境の CNAME を交換する API 呼び出しが実行されます。

! Elastic Beanstalk で Blue/Green を実行するには、新しい環境にデプロイして CNAME スワップを実行する必要があります。CNAME スワップ機能は CloudFormation 自体ではサポートされていないため、その API 呼び出しを実行し、CodePipeline のカスタムジョブの一部として呼び出すカスタム Lambda 関数を作成する必要があります。

## EC2 インスタンスに使用される新しい AMI が毎週更新されるようにしたい
- Golden AMI ID を SSM パラメータストアパラメータに保存します。SSM パラメータストアを指し、Elastic Beanstalk 環境の設定に渡される CloudFormation パラメータを作成します。
- 毎週トリガーされ、Lambda 関数を起動する CloudWatch イベントルールを作成します。この Lambda 関数は、UpdateStackAPIを使用してすべての CloudFormation テンプレートの更新をトリガーする必要があります。

## CloudTrail の有効化のコンプライアンスを追跡し、問題が発生した場合に自動的に警告を受け取れるようにしたい
- CloudFormation テンプレートを作成して CloudTrail を有効にします。StackSet を作成し、その StackSet を AWS 組織内のすべてのアカウントとリージョンにデプロイします。
- 別の CloudFormation StackSet を作成して AWS Config を有効にし、CloudTrail が有効になっているかどうかを追跡する Config ルールを作成します。集中管理されたアカウントの AWS Config アグリゲータを作成してコンプライアンスを追跡します。
- コンプライアンス違反が発生したときにイベントを生成する CloudWatch イベントを作成し、通知を送信する Lambda 関数をサブスクライブします。

! 正しいソリューションでは StackSet を活用してすべてのアカウントとすべてのリージョンで Config を有効にしなければならないこと

## AWS のすべてのリソースのタグ付けガイドラインと標準を定義しており、すべてのリソースのコンプライアンスを視覚化し、コンプライアンス違反のリソースを見つけられるダッシュボードを作成したい
- AWS Configを使用してアカウント内のリソースを追跡します。SNSを使用して、S3に書き込むLambda関数に変更をストリーミングします。その上にQuickSightダッシュボードを作成します。

## 一部のクライアントが DNS レコードに対して正しく動作していないため、非アクティブな ALB+ASG グループにリクエストを送信していることが判明しました。同社は、この動作を最小限のコストで改善し、ソリューションの複雑さを軽減した
- 1つのALBを削除し、2つのASGを維持します。新しいデプロイメントが発生した場合は、古いASGにデプロイし、ALBルールのターゲットグループを交換します。Route53レコードをALBにポイントしたままにします。

! 正しい解決策は、ロード バランサーの背後にあるインフラストラクチャのみを置き換えることです。要約すると、1 つの ALB のみに移行し、各 ASG の背後で一度に 1 つのターゲット グループを使用して正しいルーティングを行うことができます。

##  CodeDeploy を使用して、ローリング アップデートによるデプロイメント中のダウンタイムをゼロにしたいと考えています。アプリケーションがヘルス チェックに合格できないことで判断されるデプロイメントの失敗の場合には自動的にロールバックすることを望んでいます。
- ValidateServiceのフックでappspec.yml、サービスが正しく実行されていることを確認します。デプロイ失敗時にロールバックするようにCodeDeployを設定します。フックが失敗した場合、CodeDeployはロールバックします。

## API Gateway を使用して従量課金制の API を作成。 新しい API ルートに少量のトラフィックを送信して、メトリクスが正常かどうかをテストする方が安全だと判断しました。API ゲートウェイ ルートは AWS Lambda によってサポートされています
- 新しい API ゲートウェイ ステージを作成します。v1 ステージでカナリア デプロイメントを有効にします。新しいステージを v1 ステージにデプロイし、少量のトラフィックをカナリア ステージに割り当てます。CloudWatch を使用してメトリクス データを追跡します。

! 新しい API ルートを実装する場合は、Lambda エイリアスではなく、新しい API Gateway ステージを作成する必要があります

## AWS 初心者の従業員が準拠していないリソースを起動するリスクを最小限に抑えたい
- よく使用されるアーキテクチャを CloudFormation テンプレートとして定義します。これらのテンプレートから Service Catalog スタックを作成し、タグ付けが適切に行われていることを確認します。IAM ユーザーを初心者グループに配置し、ユーザーが Service Catalog からのみスタックを起動できるようにし、他のサービスへの書き込みアクセスを制限します。

! AWS Config ルールは状況を「監視」する方法ですが、リソースが間違った方法で作成されるのを防ぐことはできません。

## Elastic Beanstalk にデプロイしたいと考えています。これらの言語の一部はサポートされていますが (Node.js、Java、Golang など)、Rust などの他の言語はサポートされていません
- マルチDockerコンテナ構成を使用してElastic Beanstalkにデプロイします。各アプリケーションをECRのDockerコンテナとしてパッケージ化します。

## アプリケーションの構成には時間がかかり、現在ウォームアップに 20 分以上かかります。そのうち 10 分は Web アプリケーションのインストールと構成に費やされ、さらに 10 分はローカル インスタンス データ キャッシュのウォームアップに費やされます。向上させるには？
- ウェブアプリケーションを含むAMIを作成します。EC2ユーザーデータスクリプトを使用して実行時に動的部分を設定します。

! キャッシュは動的であり、データは時間の経過とともに変化する可能性があるため、残念ながらローカル キャッシュのウォームアップを改善することはできません。そのため、ローカル データ キャッシュのコピーを含む AMI を作成することは、邪魔になるだけです。

## 新しい AMI が作成されると、チームは古い AMI から新しい EC2 インスタンスをインスタンス化できないようにする必要があります。
- CloudFormation StackSetsを使用して、すべてのアカウントでAWS Configカスタムルールを作成します。AWS Config集計を使用してルールの結果を報告します。
- AWS Automationドキュメントを作成して、マスターアカウントでAMIを作成し、他のアカウントとAMIを共有します。新しいAMIが作成されたら、以前のAMIの共有を解除し、新しいAMIを共有します。

## デプロイごとに新しい Lambda 関数が少量のトラフィックを 10 分間維持し、その後すべてのトラフィックを新しい関数に移行することを決定しました。また、Lambda 関数がクラッシュを何度も経験した場合に自動的にロールバックする安全策
- 展開構成を選択するLambdaCanary10Percent10Minutes
- Lambda CloudWatchメトリクスにCloudWatchアラームを作成し、CodeDeployデプロイメントに関連付けます。

! カナリア デプロイメントとは、LambdaCanary10Percent10Minutes10 分間はトラフィックの 10% が新しい機能に集中し、時間が経過するとすべてのトラフィックが新しいバージョンに移行することを意味します。

## CSV 形式でアップロードした請求書を処理する Elastic Beanstalk に主力アプリをデプロイしました。請求書は最大 10 MB で、合計 1,000,000 件のレコードと大きくなります。処理には CPU が集中するため、アプリの速度が低下します。処理が完了すると、cron ジョブを使用して顧客にメールが送信されます。監査会社は、この要件に対応するソリューションを構築
- ワーカー環境である別のBeanstalk環境を作成し、SQSキューを介して請求書を処理します。
- 請求書はS3にアップロードされ、その参照がWeb層によってSQSに送信されます。ワーカー層はこれらのファイルを処理します。cron.ymlファイルを使用して定義されたcronジョブは、電子メールを送信します。

! cron.ymlファイルはワーカー層で定義する必要があります。

## この会社は、バックエンドで 1 つの Lambda 関数のみを管理し、新旧両方のクライアントをサポートできるようにしたいと考えています。API Gateway API は現在ステージにデプロイされていますv1。古いクライアントには Android アプリケーションが含まれており、更新に時間がかかる場合があります。技術要件では、ソリューションが今後何年も古いクライアントをサポートすることが義務付けられています。
- 新しい Lambda 関数バージョンを作成してリリースします。新しい API Gateway ステージを作成し、ステージにデプロイします。どちらも、ステージとステージのv2バッキングルートとして同じ Lambda 関数を使用します。ルートに静的マッピングを追加して、リクエストに追加します。v1v2v1"color": "none"

## 、CodeDeploy を使用して EC2 インスタンスに主力アプリケーションをデプロイし、RDS PostgreSQL データベースを使用してデータを保存し、DynamoDB を使用してユーザー セッションを保存しています。この企業の主任 DevOps エンジニアとして、アプリケーションが RDS と DynamoDB に安全にアクセス
- RDS 認証情報を Secrets Manager に保存し、EC2 が Secrets Manager と DynamoDB にアクセスするための IAM インスタンス ロールを作成します。

! Secrets Manager は DynamoDB をサポートしていない

## Elastic BeanstalkにNode.jsアプリケーションをデプロイしています。エラー率を追跡し、具体的には、アプリケーションログを見て、5分間隔で100件を超えるエラーが発生していないことを確認する必要があります。エラーが多すぎる場合は、電子メールで警告を受け取りたい
- CloudWatch Logs メトリクスフィルターを作成し、CloudWatch メトリクスを割り当てます。メトリクスにリンクされた CloudWatch アラームを作成し、SNS をターゲットとして使用します。SNS でメールサブスクリプションを作成します。

! Elastic Beanstalk Health Metrics は、ログ ファイルに送信されたエラーを追跡しないため、ユースケースの要件を満たしません。

## CloudFormation テンプレートは、S3 バケットと、S3 にアップロードされた画像をサムネイルに変換する Lambda 関数を作成します。これらの機能テストのクリーンアップの一環として、CloudFormation スタックが削除されますが、現時点では削除は失敗しています。
- S3 バケットにはファイルが含まれているため、CloudFormation では削除できません。バケットのクリーンアップを実行する Lambda 関数によってサポートされる追加のカスタム リソースを作成します。

## Auto Scaling (ASG) の背後に複数の EC2 インスタンスがあり、インスタンスが終了する前にそのインスタンス内のすべてのログ ファイルを取得、すべてのログ ファイルのメタデータ インデックスを構築して、インスタンス ID と日付範囲で効率的に検索できるようにしたいと考えています。
- ASG の終了フックを作成し、AWS Lambda 関数をトリガーする CloudWatch イベント ルールを作成します。Lambda 関数は SSM 実行コマンドを呼び出して、EC2 インスタンスから S3 にログ ファイルを送信します。
- インスタンスIDの主キーと日時のソートキーを持つDynamoDBテーブルを作成する
- S3イベントによってトリガーされるLambda関数を作成しますPUT。DynamoDBテーブルに書き込みます。

## CodeCommit マスター ブランチのコードが自動的に Docker コンテナとしてパッケージ化され、ECR に公開されるように、CICD パイプラインを作成しています。その後、チームは、そのイメージが Blue/Green 戦略を使用して ECS クラスターに自動的にデプロイされるようにしたい
- CodeBuild ステージを呼び出す CodePipeline を作成します。CodeBuild ステージは、CLI ヘルパーを使用して ECR 認証情報を取得し、イメージをビルドして、ECR にプッシュする必要があります。CodeBuild ステージが成功したら、ECS サービスをターゲットとして CodeDeploy ステージを開始します。

## API はデータ ペイロードを DynamoDB に保存し、静的コンテンツは S3 を通じて提供されます。分析を行ったところ、読み取りリクエストの 85% がすべてのユーザー間で共有されていることがわかりました。コストを削減しながらアプリケーションのパフォーマンスを向上させる
- DynamoDB の DynamoDB Accelerator (DAX) と S3 の CloudFront を有効にする

! Redis を DynamoDB と統合することはできますが、DAX を使用するよりもはるかに複雑ですが、DAX の方がはるかに適しています。

## ビルド環境の管理の手間を省きたいと考えています。同社の DevOps チームは、すべてのビルド タスクに CodeBuild を使用し、CodeBuild によって作成された成果物にテスト対象のブランチに基づいて名前を付けたいと考えています
- buildspec.yml実行時に環境変数を検索するファイルを作成します。ファイルのセクションCODEBUILD_SOURCE_VERSIONで変数を使用します。artifactsbuildspec.yml

! 特定のユースケースでは、環境変数を使用する必要があります。変数はCODEBUILD_SOURCE_VERSION実行時に CodeBuild 内で直接公開され、CodeCommit でテストされるコードのブランチ名を表します。これが最適なソリューションです。

## CloudFormation テンプレートを使用して Lambda 関数をデプロイ。Lambda 関数のコードは、s3://my-bucket/my-lambda-code.zip必要なビルド チェックをすべて通過した後、CodePipeline によって S3 にアップロードされます。その後、CodePipeline は CloudFormation テンプレートを呼び出して新しいコードをデプロイします。CloudFormationは正常に実行されるものの、Lambda 関数が更新されない
- 毎回新しいS3バケットにコードをアップロードする
- 毎回同じバケットに新しいファイル名でコードをアップロードする
- S3のバージョン管理を有効にし、S3ObjectVersionキーを提供する

! ここでの問題は、次のパラメータのいずれかが変更されない限り、CloudFormation が新しいファイルが S3 にアップロードされたことを検出しないことです: - S3Bucket - S3Key - S3ObjectVersion

## AWS アカウント内のユーザー、アプリケーション、SDK によって行われたすべての API 呼び出しを追跡するソリューションを導入
- AWS CloudTrailを使用してAPI呼び出しのログ記録をオンにします。ログをS3バケットに配信し、ログ検証整合性API呼び出しを使用してログファイルを検証します。

## DeleteTableゲーム会社は、DynamoDB でAPI 呼び出しが呼び出されたときに、ほぼリアルタイムの通知を受信できるようにしたいと考えています。
- CloudTrail を有効にします。CloudTrail 経由で AWS API 呼び出しを追跡し、SNS をターゲットとして使用する CloudWatch イベント ルールを作成します。

! CloudTrail ログを CloudWatch Logs に送信し、それらにフィルターを作成することは機能しますが、単一のイベントを抽出するためだけに CloudTrail からすべてのログをストリーミングするため、コストがかかります。

## 、最近、一部の EBS ボリュームが暗号化されていないことが判明しました。会社は、コンプライアンスを継続的に監視および監査し、暗号化されていない EBS ボリュームが検出された場合は、対応するチームに警告する必要があります。
- EBSボリューム暗号化をチェックするAWS Configマネージドルールを作成します。CloudWatchイベントルールを使用してアラートを提供します。

! Config の SNS トピックは、すべての通知と構成の変更をストリーミングするためにのみ使用できます。単一のルールのアラートを分離するには、CloudWatch イベントを使用する必要があります

## ECS クラシック (EC2 上) をコンテナ オーケストレーション システムとして使用し、ECR を Docker レジストリとして使用することを決定しました。監視パイプラインの実装の一環として、すべてのアプリケーション ログを CloudWatch ログに保存できるようにする
- ドライバーを含むECSタスク定義を作成しますawslogs。CloudWatchログへの書き込みに必要な権限を持つIAMインスタンスロールをEC2インスタンスに設定します。

! CloudWatch ログへの書き込みには、IAM タスクロールではなく、EC2 インスタンスロールに適切な権限を付与する必要があります。

##  EC2 インスタンスにインストールされているすべてのソフトウェア パッケージのリストを定期的に生成するように依頼しました。ソリューションは、AWS アカウントの将来のインスタンスに拡張でき、インスタンスがソフトウェアを追跡
- CloudWatch イベントルールを作成し、1 時間ごとに Lambda 関数をトリガーします。EC2 で実行されているインスタンスと SSM によって追跡されているインスタンスを比較します。
- インスタンスに SSM エージェントをインストールします。SSM インベントリを実行してメタデータを収集し、Amazon S3 に送信します。

## Web アプリを Auto Scaling Group にデプロイしています。チームは、インスタンスが ASG で小さなバッチで更新されるように、ローリング アップデートの戦略を選択しました。デプロイの終了時に、ASG では 5 つのインスタンスが稼働しています。3 つのインスタンスはアプリケーションの新しいバージョンを実行しており、他の 2 つは古いバージョンを実行しているようです。
- Auto Scalingスケールアウトイベントにより、デプロイメント中に2つの新しいインスタンスが作成されました。

! デプロイの進行中に Amazon EC2 Auto Scaling のスケールアップイベントが発生した場合、新しいインスタンスは、現在デプロイされているアプリケーションリビジョンではなく、最後にデプロイされたアプリケーションリビジョンで更新されます。

## Git リポジトリ テクノロジを採用しています。コードの運用ブランチが運用環境にデプロイされていることを確認したいだけでなく、ユーザー受け入れテストを実行するために、他のバージョンを開発環境とステージング環境にデプロイできるようにしたいと考えています。
- CodeCommit リポジトリを作成し、マスター ブランチへの変更を開発環境とステージング環境にデプロイする CodePipeline パイプラインを作成します。コードがプル リクエストを通じてマージされた後、本番ブランチへの変更を本番環境にデプロイする 2 番目の CodePipeline パイプラインを作成します。

! 2 つのバージョンのコードを異なる環境にデプロイする必要があることです。そのため、2 つの CodePipeline が必要になります。

## Elastic Beanstalk に Spring アプリケーションをデプロイ。環境変数を介して RDS PostgreSQL を参照し、アプリがデータを保存。スキーマが変更された場合に備えて、ライブラリを使用してデータベースの移行を実行。Beanstalk に更新をデプロイすると、新しいバージョンを実行しているすべての EC2 インスタンスが RDS データベースで移行を実行しようとしたため、移行が失敗しました。
- .ebextensions/db-migration.configコードリポジトリにファイルを作成し、container_commandsブロックを設定します。そこに移行コマンドを設定し、leader_only: true属性を使用します。

## AWS CodePipeline によってオーケストレーションされた CICD パイプラインを使用してグローバル アプリをデプロイしています。パイプライン現在 eu-west-1 に設定されており、拡張してアプリを us-east-2 にデプロイしたいと考えています。これには、マルチステップの CodePipeline をそこに作成して呼び出す必要があります。
- eu-west-1のパイプラインの最後に、CodeDeployで使用されている成果物をus-east-2のS3バケットにコピーするためのS3ステップを含めます。S3からus-east-2のCodePipelineソースファイルを作成します。

## 共通脆弱性識別子 (CVE) プログラムを通じて新たに公開された脆弱性に基づいて、AMI の脆弱性を毎日チェックできる必要があります
- 毎日のスケジュールで CloudWatch イベントを作成し、ターゲットを Step Function にします。Step Function は AMI から EC2 インスタンスを起動し、CheckVulnerabilities: True でタグ付けします。
- その後、Step Function は AWS Inspector と上記のタグを使用して AMI 評価テンプレートを開始します。その後、インスタンスを終了します。

! 最もコスト効率が高く、最も混乱の少ない評価方法は、その目的のために AMI から EC2 インスタンスを作成し、評価を実行してから最後にインスタンスを終了することです。Step Functions は、CheckVulnerabilities: True のタグが付けられたインスタンスをターゲットにして、そのワークフローを調整するのに最適です。

## S3 バケット内の個人識別情報 (PII) データのセキュリティを最大限に高めることに注力しており、コンプライアンス要件の一環として、S3 で新しい PII とそのアクセスが発生した場合に警告するソリューションを実装する必要
- 選択したS3バケットでAmazon Macieを有効にします。CloudWatch Eventsを使用してアラートを設定します。

! Amazon Macie は、機械学習を使用して AWS 内の機密データを自動的に検出、分類、保護するセキュリティサービスです。
! Amazon Lambda + Sagemager は機能する可能性がありますが、かなりの開発作業が必要であり、優れた結果は得られないでしょう。

## 異なる大陸にある 2 つの異なるリージョンの Amazon S3 にデータを保存する必要があります。データは 1 秒あたり 10,000 オブジェクトという高速で書き込まれます。規制上の理由から、データは転送中および保存中に暗号化する必要もあります
- バケットポリシーを作成して、次のリクエストを拒否する条件を作成します"aws:SecureTransport": "false"。SSE-S3を使用して保存中のオブジェクトを暗号化します。クロスリージョンレプリケーションをセットアップします。

! KMS を使用して暗号化すると、1 秒あたり 10,000 オブジェクトに制限される可能性があります。この場合、SSE-S3 の方が適しています。

## Amazon Aurora データベースに接続し、スタック全体は現在米国に導入されています。この会社は、事業をヨーロッパとアジアに拡大する計画を立てています。テーブルはmoviesグローバルにアクセスできるようにする必要がありますが、テーブルは地域限定にする必要がありusersますmovies_watched。
- テーブルにはAmazon Aurora Global Databaseを使用し、テーブルmoviesにはAmazon Auroraを使用します。usersmovies_watched

! このユースケースでは、グローバル テーブル (movies テーブル) 用とローカル テーブル (users テーブルと movies_watched テーブル) 用の 2 つの Aurora クラスターが必要になります。

## アプリケーションは ASG で実行されており、ALB ヘルス チェック統合がオンになっています。アプリケーションでバックエンド データベースへの接続に問題が発生し、Web サイトのユーザーは障害のあるインスタンスを介して Web サイトにアクセスする際に問題が発生
- ヘルスチェックを強化して、戻りステータスコードがデータベースへの接続に対応するようにします。

## CICD パイプラインの一環として、最新のアプリケーション コードをステージング環境にデプロイし、本番環境にデプロイする前に自動化された機能テスト スイートを実行できることを確認したいと考えています。コードは CodeCommit によって管理されます
- CodeCommit リポジトリのマスター ブランチを指す CodePipeline を作成し、CodeDeploy を使用してステージング環境に自動的にデプロイします。
- そのステージの後、テスト スイートを実行する CodeBuild ビルドを呼び出します。ステージが失敗しない場合は、最後のステージでアプリケーションが本番環境にデプロイされます。

! テスト スイートを実行するために CodeBuild ビルドを使用する必要がありますが、CodeBuild を実行する前にまずステージングにデプロイする必要があります。

## 複数のシャードで構成された Kinesis Data Streams からレコードを読み取るために、KCL を使用するストリーム処理アプリを導入しました。アプリは 1 つの EC2 インスタンスで実行されています。消費側アプリケーションが大きな負荷で遅れているため、レコードが時間内に処理されず、最終的にストリームから削除されているようです。
- アプリケーションをAuto Scalingグループで実行し、CloudWatchメトリックに基づいてスケールするMillisBehindLatest
- ストリームデータの保持期間を延長する

! ストリームの保持期間は、作成後 24 時間にデフォルトで設定されています。レコードが削除されないようにするには、ストリームの保持時間を長くして、レコードを処理する余裕を持たせることをお勧めします。設定できる最大保持期間は 7 日間です。

## 開発者を IAM グループに追加しdevelopers、AWS 管理の IAM ポリシーをarn:aws:iam::aws:policy/AWSCodeCommitPowerUserグループにアタッチしました。AWS CodeCommit リポジトリへのフルアクセスを提供しますが、リポジトリの削除は許可しません。ただし、開発者は引き続きマスターブランチにプッシュできます。開発者がマスター ブランチにプッシュするのを防ぐには
- グループにアタッチされた新しいIAMポリシーを追加して、codecommit:GitPushマスターブランチで条件付きで拒否します。

! AWS 管理 IAM ポリシーは変更できない

## CodePipeline パイプラインの失敗をすべて会社の #devops Slack チャンネルに送信したいと考えています。
- 対応するソースを持つCloudWatchイベントルールを作成します。

{
  "source": [
    "aws.codepipeline"
  ],
  "detail-type": [
    "CodePipeline Pipeline Execution State Change"
  ],
  "detail": {
    "state": [
      "FAILED"
    ]
  }
}

- ルールのターゲットは、サードパーティのSlackウェブフックを呼び出すLambda関数である必要があります。

## 認証情報が漏洩した場合に警告を発し、その認証情報を使用して最近行われた API 呼び出しのレポートを生成し、認証情報を非アクティブ化するワークフローを実装したいと考えています
- ヘルスサービスで AWS_RISK_CREDENTIALS_EXPOSED をチェックする CloudWatch イベントを作成します。IAM、CloudTrail、SNS への API 呼び出しを発行して、必要な要件を満たす Step Function ワークフローをトリガーします。

! IAM アクセスキーが GitHub で公開されると、AWS Health は AWS_RISK_CREDENTIALS_EXPOSED イベントを生成します。

## Elastic Beanstalk を使用してアプリをデプロイしています。CodePipeline パイプラインのデプロイ ステージを使用してデプロイされます。技術要件では、HTTP から HTTPS へのリダイレクト ルールを追加して、Elastic Beanstalk に関連付けられた Application Load Balancer の設定を変更することが義務付けられています。
- .ebextensions/alb.configコードリポジトリにという名前のファイルを作成し、option_settingsキーのルールを指定するブロックを追加しますaws:elbv2:listener:default。コードをプッシュしてCodePipelineを実行します。

## Auto Scaling グループを更新して、すべてのインスタンスが新しく作成された起動構成を参照するようにし、インスタンス タイプをアップグレードします。現在、ASG には 6 つのインスタンスが含まれており、常に少なくとも 4 つのインスタンスが稼働している必要があります。
- 自動スケーリングローリングアップデート

## EC2 インスタンスが 24 時間以上使用されていない場合に警告する自動化ソリューションを作成したい
- Trusted Advisor を有効にし、使用率の低い EC2 インスタンスのチェックがオンになっていることを確認します。Trusted Advisor によって作成されたイベントを追跡する CloudWatch イベントを作成し、そのイベントのターゲットとして Lambda 関数を使用します。
- Lambda 関数は、手動承認手順を含む SSM 自動化ドキュメントをトリガーする必要があります。承認されると、SSM ドキュメントはインスタンスの終了に進みます。

## CloudWatch メトリクスを使用して AWS サービスとアプリケーションのメトリクスをキャプチャしています。規制要件により、これらのメトリクスを視覚化するには、最大 7 年前までさかのぼる必要があります。
- CloudWatch メトリクス ストリームを作成し、Kinesis Firehose Firehose 配信ストリームに送ります。
- Firehose を使用してすべてのメトリクス データを S3 に送信し、メトリクスを視覚化するための QuickSight ダッシュボードを作成します。Athena を使用して特定の時間範囲をクエリします。

## CodePipeline を使用して CodeCommit から CodeDeploy でコードをデプロイしたいと考えています。
- CodeDeploy がデプロイされている S3 バケットへのアクセスを許可する IAM ロールを持つ EC2 インスタンスを作成します。
- EC2 インスタンスにも CodeDeploy エージェントがインストールされていることを確認します。インスタンスにタグを付けて、デプロイ グループの一部にします。

## DevOps エンジニアとして、これをエンド カスタマー向けの安全な API として公開する必要があります。Web サイトは同時に 5000 件のリクエストに対応できる必要があります。これをできるだけ簡単に実装するにはどうすればよいでしょうか。
- Step Functions で予約ワークフローを作成します。Step Functions とのサービス統合を使用して API Gateway ステージを作成します。Cognito を使用して API を保護します。

! Step Functions を使用して支払いワークフローを実装する必要があります。この統合が必要な主な理由は、AWS Lambda の最大同時実行数が 1000 であるのに対し、API Gateway の最大同時実行数は 10000 であるためです。

## すべての Amazon Elastic Block Store (Amazon EBS) ボリュームを毎週バックアップすることを決定しました。この変更を実装するには、開発者にすべての Amazon EBS ボリュームにカスタムタグを付ける義務があります。
- AWS アカウントで AWS Config を設定します。AWS::EC2::Volumeカスタムタグが EBS ボリュームに適用されていない場合はコンプライアンス違反を返すリソースタイプのマネージドルールを使用します。
- カスタム AWS Systems Manager Automation ドキュメント (ランブック) を使用して、事前定義されたバックアップ頻度でカスタムタグをすべての非準拠 EBS ボリュームに適用する修復アクションを設定します。

## pplication Load Balancer (ALB) の背後にある Amazon EC2 インスタンスでホストされています。CodeDeploy Blue/Green デプロイメントを使用して新しいバージョンをデプロイしているときに、AllowTrafficライフサイクル イベント中にデプロイメントが失敗しました
- この問題の原因は、アプリケーションロードバランサー（ALB）のヘルスチェックの設定が間違っていることです。

! デプロイメント中にインスタンスが終了しても、デプロイメントは最大 1 時間失敗しません。

## 
