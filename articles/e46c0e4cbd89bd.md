---
title: "Glueジョブ実行履歴取得"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, glue, python, boto3]
published: true
publication_name: "iret"
---

## このページについて
AWS Glueのコストが気になるため、ジョブごとのコストを確認したく、そのために必要な情報をcsvに出力するスクリプトをPythonで作成しました。

## AWS Glue Studio
AWS Glue Studioの `Monitoring` ページで、ジョブ実行状況を確認することはできます。
ただ、いくつか制限があります。

- 表示対象の日付指定が30日間まで
- ジョブ実行履歴の件数が1,000件まで

https://docs.aws.amazon.com/ja_jp/glue/latest/ug/what-is-glue-studio.html

## boto3
Pythonスクリプトを作成し、boto3で情報取得することにしました。

使用したGlueのAPI

- get_job
https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html#Glue.Client.get_job

- get_job_runs
https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/glue.html#Glue.Client.get_job_runs

## csv出力する項目

| 項目 | 内容 |
| ---------------------- | -------------------------------------------------------------------------------------------- |
| JobId | GlueジョブID |
| JobName | Glueジョブ名 |
| StartedOn | Glueジョブ開始時間 |
| CompletedOn | Glueジョブ完了時間 |
| ExecutionTime(s) | Glueジョブの実行でリソースを消費した時間(秒) |
| ExecutionTimeRollup(s) | 最小課金処理を実施したExecutionTime |
| MaxCapacity | Glueジョブの実行に割り当てられるGlueデータ処理ユニット(DPU)の数 |
| Cost | 料金(MaxCapacity × ExecutionTimeRollup(s) / 3600 × 0.44 ) <br>https://aws.amazon.com/jp/glue/pricing/ |
| Job_Type | ジョブタイプ(SparkジョブまたはPython Shellジョブ) |
| GlueVersion | Glueのバージョン(バージョンによりGlue がサポートするSparkとPythonのバージョンを決定する) |
| Timestamp | スクリプト開始時刻 |

## Glueのコスト計算について

基本的な料金計算式

```
DPU(MaxCapacity) × Execution Time(h) × 0.44
```

GlueStudioで表示される `DPU hours` は、`DPU(MaxCapacity) × Execution Time(h)` と同じとなります。

また、ジョブタイプによって最小課金単位が変わってくるため、それも考慮します。

| ジョブタイプ | AWS Glueバージョン | 最小課金単位 |
| --- | --- | --- |
| Apache Spark または Spark ストリーミングジョブ | 0.9, 1.0 | 最小10分 |
| Apache Spark または Spark ストリーミングジョブ | 2.0以降 | 最小1分 |
| Python シェルジョブ | - | 最小1分 |

https://aws.amazon.com/jp/glue/pricing/

## 作成したPythonスクリプト

https://github.com/isseeeeey55/resource_usage

## 参考記事

https://qiita.com/suzuki-navi/items/0d273454af382fad3ad6
