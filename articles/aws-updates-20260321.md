---
title: "【AWS】2026/03/21 のアップデートまとめ"
emoji: "📝"
type: "tech"
topics: ["aws", "update"]
published: false
---

## はじめに

2026年3月21日のAWSアップデートでは、4件の機能強化が発表されました。特に注目すべきは、AWS NeuronがAmazon EKSでDynamic Resource Allocation（DRA）をサポートしたことで、機械学習ワークロードのリソース管理が大幅に改善される点です。また、AWS DataSyncでのSecrets Manager対応やAmazon RedshiftのIAM Identity Center連携強化など、セキュリティと運用性の向上に焦点を当てたアップデートが目立ちます。

## 注目アップデート深掘り

### AWS NeuronがEKSでDynamic Resource Allocation（DRA）をサポート

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
| [AWS Firewall Manager](https://aws.amazon.com/about-aws/whats-new/2026/03/aws-firewall-manager-launches-ap-nz/) | アジアパシフィック（ニュージーランド）リージョン対応 | 複数アカウント・リージョンにまたがるセキュリティポリシーの統一管理がニュージーランドリージョンで利用可能に |
| [AWS DataSync](https://aws.amazon.com/about-aws/whats-new/2026/03/aws-datasync-secrets-manager/) | 全ロケーションタイプでSecrets Manager対応 | すべてのファイルシステムとの連携において、認証情報をSecrets Managerで安全に管理可能 |
| [AWS Neuron](https://aws.amazon.com/about-aws/whats-new/2026/03/neuron-eks-dra-support/) | EKSでDynamic Resource Allocation対応 | TrainiumインスタンスのリソースをKubernetesネイティブな方法で動的に割り当て、ML ワークロードの効率化を実現 |
| [Amazon Redshift](https://aws.amazon.com/about-aws/whats-new/2026/03/redshift-federated-permissions-idc-mrr/) | IAM Identity Center連携マルチリージョン対応 | 複数リージョンのRedshiftクラスターに対して統一されたアクセス管理が可能 |

## まとめ

今回のアップデートは、特に大規模システムの運用効率化とセキュリティ強化に焦点を当てた内容となっています。Neuron DRAサポートは機械学習基盤の運用を劇的に改善する可能性を秘めており、DataSyncのSecrets Manager対応はデータガバナンスの観点で重要な前進です。

全体的な傾向として、AWSは単純な機能追加よりも、既存サービス間の連携強化や運用負荷軽減に注力していることが伺えます。特に、KubernetesやIAM Identity Centerのようなエコシステム全体を意識したアップデートが増えており、これらを適切に活用することで、より効率的で安全なクラウドインフラの構築が可能になるでしょう。