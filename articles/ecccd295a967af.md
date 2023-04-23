---
title: "[AWS Summit Tokyo 2023] AWS Jam体験記"
emoji: "🤘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [aws, awssummit, イベントレポート]
published: true
---

## このページについて
今回久しぶりにオフラインでAWS Summit Tokyoに行ってきました。
そしてこの日、AWS Jamのみ参加してきましたので、その体験記を残しておきます。
チャレンジ内容自体はブログに書くことができないのでご了承ください。

## AWS Jamとは

AWS Jamとは、AWSが主催するワークショップ形式のイベントです。AWSの様々なサービスを実際に手を動かしながら学ぶことができるため、参加者はAWSのサービスや開発手法をより深く理解することができます。

AWS Jamでは、複数のチャレンジが設けられており、各チャレンジではAWSのサービスを使用したハンズオン形式のワークショップが行われます。例えば、チャレンジごとにEC2、S3、Lambda、API Gatewayなど、AWSの様々なサービスを扱った内容となっています。また、AWS Jamでは、実際にコードを読み書きしながら、AWSの機能を体験することができるため、理論だけでなく実践的なスキルも身につけることができます。

### 今回の開催概要
https://aws.amazon.com/jp/blogs/news/aws-summit-tokyo-2023-aws-jam-intro/

## 全体の流れ
```
12:20 – 12:50: 受付
12:50 – 13:50: AWS Jam のアカウント作成と事前説明
13:50 – 17:05: AWS Jam 実施
17:05 – 17:50: 結果発表、表彰など
17:50 – 18:00: アンケート
```

なんと午後いっぱい使います！
他のセッションなど参加することはできません！！
集中してチャレンジします！！！

## 顔合わせ
受付でくじを引き、くじに書かれたチームのテーブルに向かいます。
今回は各チーム4名で編成されました。

はじめの自己紹介では、普段の業務でどんなことやってるか、取得したAWS認定資格や好きなAWSサービスなどを含めて自己紹介するので、こんな時のために日頃から好きなAWSサービスを意識しておくと良いと思います。
私は今回「CloudFormation」でいきました！

久しぶりのオフラインイベントでいきなり知らない方々と顔合わせるのはドキドキ強めでしたが、みなさん普段からAWSを触っていることもあり、AWSが共通言語となり、わいわい雑談できました。

## チャレンジ内容
実際チャレンジした内容は書けないのですが、レベルは「Easy/Medium/Hard」と3段階に分かれており、全部で14問。
Easyの問題はAWS初心者の方でも解けるようなチャレンジもあり、どんなレベルの方でも参加できるイベントだと思いました！ちなみに、Hardは2問でした。
チャレンジ中にググったりして調査するのはOKです。

ただ、今回のAWS Jamの特徴としては「モブワークもしくはペアワーク」が必須ということでした。つまり、ソロ(一人)で作業することは禁止されていました。
また、ドライバ(作業者)のPCだけを使い、ナビゲータ(フォローする人)は自分のPCで調査したりなどの作業は禁止です。

### 自分がドライバの場合
黙々と作業してしまうと相手に何をやっているのか伝わらないため、どういう目的で今この操作をしているのか言葉に出しながら作業する必要があります。
また、普段から使っているショートカットなどがあれば、それについても多少説明しないと、相手にとっては？となってしまいますので注意です。

### 自分がナビゲータの場合
普段自分が調査している流れ(ググり方など含めて)、言語化して相手に伝えないといけません。
これがなかなか難しくて、相手が操作している途中に口を挟んで指示していくことも必要なので、そのタイミングやどういう目的で調査した方が良いなど伝え方に気をつけないとうまく進みません。
逆に、相手の作業を見て、「こんな流れで作業するのか！」「こんなショートカットがあるのか！」といった気づきも得られます。

### 私のチームの進め方
基本的にペアワークでチャレンジを行い、途中でヒントが必要になった際は一度チームで会話し、その上でヒントを使おうという方針にしました。(Clueと呼ばれる手掛かりがあり、正解して得られるポイントが減る代わりにヒントを得られます)

途中、やはりヒントが必要になるシーンは出てくるのですが、事前の会話通りチーム内全員でコミュニケーションを取り、モブワークの形を取り、テーブルに用意されていたホワイトボードも駆使して進めることもありました。
そのチャレンジでは最終的に、それでもヒントを使ったのですが、良い形でモブワークを体験できたと思いました。

## 最終結果
私のチームは途中1位になることもあったのですが、その後とある問題でハマりにハマってしまい失速。。
最終的には6位という結果でした。

こちらのスコアリングトレンドが会場の前に映し出され、リアルタイムで順位が変動したことを実況してくれていて盛り上がりました！！
![](/images/ecccd295a967af/awsjam_scoring_trend.jpg)

## 振り返り
1位には記念トロフィーが授与されるということで、頑張ってスコアを取ろうと躍起になりがちですが、今回のAWS Jamでは「モブワークもしくはペアワーク」を行うことで、AWS以外のことも多く学ぶことができたと思いました。

ちなみに、集中しすぎてトイレにも行かずに頑張ってましたw
が、終わった後の今考えると、休憩は計画的に取った方が頭も働くかなと思いました。

普段からリモートワークの私ですが、あまり画面を共有しながら作業することは多くありません。
作業した後に、別の方に作業途中のエビデンスや作業結果を確認してもらう流れが多いため、自分の作業を言語化しながら進めるということの難しさや重要さを改めて思い知りました。結局コミュニケーション大事ということですね。
もちろん、AWSの知識としてもチャレンジする中で知らなかった設定などありましたし、学びは大きかったと思います。

最終スコアが発表された後には、チームで振り返る時間が用意されていて、今回のチャレンジした内容で難しかった内容や学びの得られたことなど共有できました。
また、全てのチャレンジ内容の解答を解説する時間はありませんでした。
数問の解き方の流れを説明するくらいだったかなと思います。

## 最後に
想像以上に楽しかったので、とてもオススメできるイベントでした！
このイベントはオフラインでこそ味わえる楽しさがありますね。
またチャレンジできる機会があれば参加したいです！！

優勝されたチームの方々、おめでとうございます！
参加されたみなさん、ありがとうございました！！
https://twitter.com/awscloud_jp/status/1648977004644167680?s=20
