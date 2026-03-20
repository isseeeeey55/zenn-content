# Zenn Content

[Zenn](https://zenn.dev/isseeeeey55) に公開する技術記事を管理するリポジトリ。
GitHub 連携により、`main` ブランチへの push で自動デプロイされる。

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

## 記事の作成

```bash
npx zenn new:article
```

## 関連リンク

- [Zenn プロフィール](https://zenn.dev/isseeeeey55)
- [Zenn CLI ガイド](https://zenn.dev/zenn/articles/zenn-cli-guide)
