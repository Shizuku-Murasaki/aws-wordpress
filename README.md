# AWS上にマルチAZでWordPressサイトを構築

## [WordPress](https://www.awsexample.com)

## 実現したかったこと
* ### インターネットから暗号化された通信を使ってドメイン名でアクセスできること
* ### マルチAZ構成でWebサーバーを稼働し、高可用性を実現すること。
* ### マルチAZ構成でデータベースサーバーを稼働し、高可用性を用意すること
* ### ストレージやデータベースのデータに不正にアクセスされても、解読できないように暗号化されていること
* ### EC2に障害が起きた時は通知をだし、自動的に復旧すること
* ### EBSの使用率や、ALBとEC2の死活監視をおこない、閾値を超えた時は通知を出すこと
* ### apacheのアクセスログをマネジメントコンソールから確認できるようにし、それをコストの低い保管方法で保存すること

## AWS構成図
![AWS構成図_Wordpress](https://user-images.githubusercontent.com/102236945/167352761-90bd6feb-7c8b-4eda-9b15-40a0f85a9b8b.png)

## 使用したサービス
| サービス名 | 利用目的 |
|:---|:---|
|Route53 |名前解決のため |
|ACM（AWS Certificate Manager）|https通信を行うための証明書の作成/利用のため |
|ALB (Application Load Balancer) |二つのEC2にヘルスチェックを行いながらリクエストを振り分け、高可用性を実現するため |
|IAM (Identity and Access Management) |EC2にCWAgent用の権限を与えるため |
|KMS (Key Management Service) |RDS、EBSなどの暗号化のために暗号化鍵の作成と管理を行うため |
|AWS Systems Manager |CloudWatchAgentの設定パラメータを配置するため |
|EC2 (Elastic Compute Cloud) |Webサーバーとして使用するため |
|RDS (Relational Database Service) |DBサーバーとして使用するため |
|EFS (Elastic File System) |WordPressのインストールファイルをEC2間でファイル共有を行うため |
|S3 (Simple Storage Service) |Apacheアクセスログのエクスポート先として使うため |
|CloudWatchAlarm |・システムエラー、EC2とALBの死活状態、ディスク使用率の監視を行うため<br>・閾値を超えればメールで通知し、システムエラーが起きた時にオートリカバリーをするため |
|CloudWatchAgent |・ディスク使用率の監視メトリクスをCWAlarmに発行するため<br>・ApacheアクセスログをCloudWatchLogsに発行するため |
|CloudWatchLogs |Apacheアクセスログの発行先 |
|Amazon EventBridge(CloudWatchEvents） |Lambdaのトリガーとして利用するため |
|Lambda |CloudWatchLogsからS3へとログをエクスポートするため |
|SNS (Simple Notification Service) |アラーム発生時のメール送信先トピックを作成するため |