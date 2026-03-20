# Zenn Content

[Zenn](https://zenn.dev/isseeeeey55) に公開する技術記事を管理するリポジトリ。
GitHub 連携により、`main` ブランチへの push で自動デプロイされる。

## 記事の種類

### 手動記事

Zenn CLI またはエディタで作成する通常の技術記事。

```bash
npx zenn new:article
```

### AWS アップデートまとめ（自動生成）

[rss-notifier](https://github.com/isseeeeey55/rss-notifier) の Zenn Publisher Lambda が毎日 8:00 JST に自動生成するドラフト記事。

- **ファイル名:** `articles/aws-updates-YYYYMMDD.md`
- **生成元:** Slack `#notify-aws-whats-new` の過去24時間の投稿
- **AI モデル:** AWS Bedrock（Claude Sonnet）
- **状態:** `published: false`（ドラフト）で push される

記事の内容確認・編集後、`published: true` に変更して push すると公開される。

## ディレクトリ構成

```
.
├── articles/          # 技術記事（.md）
├── books/             # 本（未使用）
├── images/            # 記事で使用する画像
└── package.json       # Zenn CLI
```

## ローカルプレビュー

```bash
npx zenn preview
# → http://localhost:8000 でプレビュー
```

## 関連リンク

- [Zenn プロフィール](https://zenn.dev/isseeeeey55)
- [Zenn CLI ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [rss-notifier](https://github.com/isseeeeey55/rss-notifier) - AWS アップデート記事の自動生成元
