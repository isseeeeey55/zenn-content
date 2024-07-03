---
title: "Lambdaのトリガー上限数について"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, lambda, awslambda, quota]
published: true
publication_name: "iret"
---

## このページについて
Lambdaのトリガーを複数設定する場面があり、トリガーの上限数があるのか確認した時のメモです。

![](/images/ac8c5dd75bcca6/lambda_trigger.png)


## Lambdaクォータ
AWSドキュメントを確認しますが、Lambdaクォータに `トリガー数` について記載は見つけられませんでした。
https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/gettingstarted-limits.html

### ポリシーのサイズ上限
トリガー数に関するクォータは確認できませんでしたが、 `関数リソースベースのポリシー` について記載されていました。
Lambdaトリガーを追加するとLambda関数のポリシーが自動追記されますが、それが上限20KBであるということです。

>関数リソースベースのポリシー
>20 KB
>https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/gettingstarted-limits.html

## リソースベースのポリシーの確認方法

- Lambdaの `アクセス権限` から `リソースベースのポリシー`

![](/images/ac8c5dd75bcca6/lambda_resourcebase_policy.png)

- `ポリシードキュメントを表示` とするとポリシーのJSONが出てくる。ここが延々と冗長に追記されていく。

![](/images/ac8c5dd75bcca6/lambda_policy_document.png)

### サイズ確認
以下のコマンドで確認可能です。

>jq を使用して Lambda 関数のポリシーのサイズを調べる get-policy コマンドの例

```
$ aws lambda get-policy --function-name my-function | jq -r '.Policy' | wc -c
```

https://aws.amazon.com/jp/premiumsupport/knowledge-center/lambda-resource-based-policy-size-error/
