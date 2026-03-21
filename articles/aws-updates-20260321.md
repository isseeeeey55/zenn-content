---
title: "【AWS】2026/03/21 のアップデートまとめ"
emoji: "📝"
type: "tech"
topics: ["aws", "update"]
published: true
---

## はじめに

2026年3月21日のAWSアップデートでは、6件の機能強化が発表されました。特に注目すべきは、Amazon Bedrock AgentCore RuntimeへのWebRTCサポート追加と、Amazon EKS Provisioned Control Planeの99.99% SLA・8XLティア新設です。加えて、AWS NeuronのEKSでのDynamic Resource Allocation（DRA）サポートによるMLワークロード管理の改善、AWS DataSyncのSecrets Manager対応、Amazon RedshiftのIAM Identity Center連携強化など、AIワークロード基盤とセキュリティの両面で充実した内容となっています。

## 注目アップデート深掘り

### Amazon Bedrock AgentCore RuntimeにWebRTCサポートを追加

AgentCore Runtimeに、既存のWebSocketに加えてWebRTCプロトコルが追加されました。ブラウザやモバイルアプリケーションでの音声AIエージェント構築において、ピアツーピア・UDPベースの低レイテンシー双方向ストリーミングが可能になります。

**なぜこのアップデートが重要なのか**

WebSocketはTCPベースの永続接続でテキスト・音声のストリーミングには適していますが、リアルタイムの音声・映像通信ではUDPベースのWebRTCに劣ります。これまでリアルタイム音声AIエージェントの構築には独自のメディアサーバー構成が必要でしたが、AgentCore Runtimeが直接WebRTCをサポートしたことで開発の複雑さが大幅に軽減されます。

**TURNリレーの構成オプション**

NAT越えが必要な場合、以下の3つの選択肢が用意されています：

- **Amazon Kinesis Video Streams マネージドTURN**（推奨）: フルマネージドでAWS IAM統合、追加のインフラ管理不要
- **サードパーティTURNプロバイダー**: Twilio、Xirsysなど既存のTURNインフラを活用
- **セルフホストTURN**: coturnなどを自前運用、完全なカスタマイズが可能

**対応リージョン**

東京リージョンを含む14リージョンで利用可能です（米国3、アジア太平洋5、欧州5、カナダ1）。AWSからはAmazon Nova Sonicを使った音声エージェントのサンプルや、Pipecat・LiveKit・Strands Agents SDKとの連携実装例も公開されています。

### Amazon EKS Provisioned Control Planeの99.99% SLAと8XLティア

Amazon EKS Provisioned Control Planeに対して、SLAが従来の99.95%から99.99%に引き上げられました。また、新たに8XLスケーリングティアが追加され、4XLティアの2倍のKubernetes APIサーバーリクエスト処理能力を提供します。

**なぜこのアップデートが重要なのか**

99.95%から99.99%への引き上げは、年間ダウンタイム許容量で見ると約26分から約5分への大幅な改善です。1分間隔での可用性測定が行われるため、ミッションクリティカルなワークロードの信頼性保証が強化されます。

**8XLティアのユースケース**

- 超大規模AI/MLトレーニングクラスター
- ハイパフォーマンスコンピューティング（HPC）
- 大規模データ処理パイプライン
- 数千ノード規模のクラスター運用

EKS Provisioned Control Planeが提供されている全リージョンで利用可能です。

### AWS NeuronがEKSでDynamic Resource Allocation（DRA）をサポート

:::message
**AWS Neuronとは？**
AWS Neuronは、AWSの独自AIチップ「Trainium」（学習用）および「Inferentia」（推論用）向けのSDKです。コンパイラ、ランタイム、プロファイラなどを含み、PyTorchやTensorFlowで書かれたMLモデルをこれらのチップ上で効率的に動作させるためのソフトウェアスタックを提供します。NVIDIAのCUDAに相当するAWS独自チップ向けの開発基盤と捉えるとわかりやすいでしょう。
:::

このアップデートは、機械学習ワークロードのリソース管理に革命をもたらす可能性があります。従来のKubernetesでは、GPU等の特殊なハードウェアリソースの割り当てはDevice Pluginに依存しており、細かな制御が困難でした。しかし、DRAの導入により、ハードウェアトポロジーとデバイス属性が直接Kubernetesスケジューラに公開され、より柔軟で効率的なリソース管理が可能になります。

**なぜこのアップデートが重要なのか**

大規模な機械学習ワークロードでは、複数のTrainiumチップを効率的に活用することが性能向上の鍵となります。従来の方法では、MLエンジニアがインフラの詳細を理解し、複雑な設定を行う必要がありました。DRAにより、このような複雑さが抽象化され、Kubernetesネイティブなリソース要求で適切なハードウェアを自動的に割り当てることができます。

**具体的な検証手順**

まず、EKSクラスターでNeuron DRAドライバを有効化します：

```bash
# Neuron DRAドライバのインストール
kubectl apply -f https://raw.githubusercontent.com/aws-neuron/aws-neuron-eks-samples/main/dra/neuron-dra-driver.yaml

# ResourceClassの確認
kubectl get resourceclass
```

次に、ResourceClaimTemplateを定義してMLワークロードに適用します：

```yaml
# resource-claim-template.yaml
apiVersion: resource.k8s.io/v1alpha2
kind: ResourceClaimTemplate
metadata:
  name: trainiumv1-claim-template
spec:
  spec:
    resourceClassName: trainiumv1.neuron.aws
    parametersRef:
      apiVersion: resource.neuron.aws/v1alpha1
      kind: TrainiumParameters
      name: multi-chip-params
```

```yaml
# ml-workload.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: distributed-training-job
spec:
  template:
    spec:
      resourceClaims:
      - name: trainiumv1
        source:
          resourceClaimTemplateName: trainiumv1-claim-template
      containers:
      - name: training-container
        image: your-ml-image:latest
        resources:
          claims:
          - name: trainiumv1
```

**従来の方法との比較**

従来のDevice Plugin方式では、静的なリソース定義に依存していました：

```yaml
# 従来の方法
resources:
  limits:
    aws.amazon.com/neuron: 8
```

DRA方式では、より詳細な要件を動的に指定できます：

```yaml
# DRA方式
parametersRef:
  apiVersion: resource.neuron.aws/v1alpha1
  kind: TrainiumParameters
  name: optimized-topology-params
```

**パフォーマンス測定の観点**

実際のMLワークロードでの効果を測定するには、以下のメトリクスを監視します：

```python
import boto3
import time

def measure_resource_allocation_time():
    """リソース割り当て時間の測定"""
    start_time = time.time()
    
    # Kubernetes APIでPodの状態を監視
    # リソース割り当て完了までの時間を測定
    
    allocation_time = time.time() - start_time
    return allocation_time

def compare_training_throughput():
    """学習スループットの比較"""
    # DRA使用時とDevice Plugin使用時のスループット比較
    pass
```

### AWS DataSyncでSecrets Manager対応強化

AWS DataSyncが全てのロケーションタイプでSecrets Managerをサポートするようになったことで、データ転送時の認証情報管理が大幅に改善されます。これまで平文で管理していた認証情報を、Secrets Managerで暗号化して安全に保管できるようになります。

**セキュリティ向上の具体例**

従来のDataSync設定では、認証情報をパラメータストアやハードコーディングで管理していました：

```bash
# 従来の方法（非推奨）
aws datasync create-location-smb \
    --server-hostname example.com \
    --user myuser \
    --password "hardcoded-password" \
    --subdirectory /data
```

新しい方法では、Secrets Managerから安全に認証情報を取得します：

```bash
# Secrets Managerにクレデンシャルを保存
aws secretsmanager create-secret \
    --name "datasync/smb-credentials" \
    --description "SMB server credentials for DataSync" \
    --secret-string '{"username":"myuser","password":"secure-password"}'

# DataSyncロケーション作成時にSecrets Managerを参照
aws datasync create-location-smb \
    --server-hostname example.com \
    --secrets-manager-arn "arn:aws:secretsmanager:region:account:secret:datasync/smb-credentials" \
    --subdirectory /data
```

**Terraformでの実装例**

```hcl
resource "aws_secretsmanager_secret" "datasync_credentials" {
  name = "datasync/nfs-credentials"
  description = "NFS credentials for DataSync transfer"
}

resource "aws_secretsmanager_secret_version" "datasync_credentials" {
  secret_id = aws_secretsmanager_secret.datasync_credentials.id
  secret_string = jsonencode({
    username = var.nfs_username
    password = var.nfs_password
  })
}

resource "aws_datasync_location_nfs" "source" {
  server_hostname = "nfs.example.com"
  subdirectory    = "/exports/data"
  
  secrets_manager_arn = aws_secretsmanager_secret.datasync_credentials.arn

  on_prem_config {
    agent_arns = [aws_datasync_agent.on_premises.arn]
  }
}
```

**コンプライアンス要件への対応**

この改善により、SOC 2やISO 27001等のコンプライアンス要件に対応しやすくなります。認証情報の自動ローテーションや監査ログの取得も容易になります：

```python
import boto3

def setup_credential_rotation():
    """認証情報の自動ローテーション設定"""
    secrets_client = boto3.client('secretsmanager')
    
    response = secrets_client.rotate_secret(
        SecretId='datasync/smb-credentials',
        RotationLambdaArn='arn:aws:lambda:region:account:function:rotate-datasync-credentials',
        RotationRules={
            'AutomaticallyAfterDays': 30
        }
    )
    return response
```

## SRE視点での活用ポイント

**機械学習基盤の運用改善**

Neuron DRAサポートは、ML基盤を運用するSREにとって大きな価値があります。Terraformで管理されているEKSクラスターにDRAを導入することで、MLエンジニアからの「リソースが足りない」「配置が最適化されていない」といった課題を大幅に削減できるでしょう。特に、複数のMLチームが同一クラスターを共有している環境では、リソースの競合やデッドロックを防ぐ効果が期待できます。

導入時の判断基準として、現在のTrainiumワークロードでリソース割り当ての問題が頻発している場合や、手動でのノード配置調整が必要な場面が多い場合は、積極的な導入を検討すべきです。ただし、DRAは比較的新しい機能であるため、本番導入前には十分な検証期間を設け、既存のワークロードに影響がないことを確認することが重要です。

**データ転送の安全性向上**

DataSyncのSecrets Manager対応は、データ移行プロジェクトや定期的なデータ同期タスクで威力を発揮します。CloudWatch アラームと組み合わせて、認証エラーや接続失敗を即座に検知できるようになります。また、AWS Systems Manager のメンテナンスウィンドウと連携させることで、認証情報のローテーション後に自動的にデータ転送タスクを再実行する仕組みも構築できます。

障害対応のランブックにおいても、「認証情報の期限切れによるデータ転送失敗」というシナリオに対して、Secrets Managerのバージョン管理機能を活用した復旧手順を組み込むことで、より迅速な問題解決が可能になります。特に、複数の外部システムとのデータ連携が必要な環境では、統一された認証情報管理により運用負荷を大幅に軽減できるでしょう。

## 全アップデート一覧

| サービス | アップデート内容 | 概要 |
|----------|------------------|------|
| [Amazon Bedrock](https://aws.amazon.com/about-aws/whats-new/2026/03/amazon-bedrock-webrtc/) | AgentCore RuntimeにWebRTCサポート追加 | 低レイテンシーなリアルタイム双方向ストリーミングで音声AIエージェント構築が容易に |
| [Amazon EKS](https://aws.amazon.com/about-aws/whats-new/2026/03/amazon-eks-announces-sla-8xl-scaling-tier/) | Provisioned Control Planeの99.99% SLAと8XLティア | SLA引き上げと最大規模のスケーリングティア追加でミッションクリティカルなワークロードに対応 |
| [AWS Neuron](https://aws.amazon.com/about-aws/whats-new/2026/03/neuron-eks-dra-support/) | EKSでDynamic Resource Allocation対応 | TrainiumインスタンスのリソースをKubernetesネイティブな方法で動的に割り当て、MLワークロードの効率化を実現 |
| [AWS DataSync](https://aws.amazon.com/about-aws/whats-new/2026/03/aws-datasync-secrets-manager/) | 全ロケーションタイプでSecrets Manager対応 | すべてのファイルシステムとの連携において、認証情報をSecrets Managerで安全に管理可能 |
| [AWS Firewall Manager](https://aws.amazon.com/about-aws/whats-new/2026/03/aws-firewall-manager-launches-ap-nz/) | アジアパシフィック（ニュージーランド）リージョン対応 | セキュリティポリシーの統一管理がニュージーランドリージョンで利用可能に |
| [Amazon Redshift](https://aws.amazon.com/about-aws/whats-new/2026/03/redshift-federated-permissions-idc-mrr/) | IAM Identity Center連携マルチリージョン対応 | 複数リージョンのRedshiftクラスターに対して統一されたアクセス管理が可能 |

## まとめ

今日のアップデートは、AIワークロードの実行基盤強化とKubernetesエコシステムの成熟という2つのテーマが際立つ内容でした。Bedrock AgentCore RuntimeのWebRTCサポートはリアルタイム音声AIエージェントの実用性を高め、EKSの99.99% SLAと8XLティアはミッションクリティカルなコンテナワークロードの信頼性を新たな水準に引き上げます。

Neuron DRAドライバーによるMLワークロードのリソース管理改善と、DataSyncのSecrets Manager統合による認証情報管理の一元化も、日々の運用を着実に効率化するアップデートです。全体として、AWSがAI時代の要求に応える形でコンピューティング基盤を急速に進化させていることが改めて実感できるラインナップとなりました。