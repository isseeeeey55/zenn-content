---
title: "Claude Codeにスマホから指示を出す — Telegram × Zellij × Ghosttyで実現するリモート開発体験"
emoji: "📱"
type: "tech"
topics: ["claudecode", "telegram", "zellij", "ghostty", "ai"]
published: true
---

## はじめに

Claude Codeに **Channels** という新機能が追加されました。外部のメッセージングサービスから、実行中のClaude Codeセッションにリアルタイムでメッセージを送り込むことができる機能です。

本記事では、Telegramを使ってスマホからClaude Codeに指示を出す環境を構築する方法を紹介します。ターミナルエミュレータにはGhostty、ターミナルマルチプレクサにはZellijを使い、PCから離れていてもAIに作業を依頼できる構成を実現します。

## Claude Code Channelsとは

### 概要

**Channels**は2026年3月20日にリサーチプレビューとして公開された機能で、Claude Code **v2.1.80以降**で利用できます。

従来のClaude Codeは、ターミナルの前に座って対話する使い方が基本でした。Channelsを使うと、TelegramやDiscordなどの外部サービスから実行中のセッションにメッセージをプッシュし、Claude Codeがそれに応答できるようになります。

### アーキテクチャ

Channelsの実体は **MCP（Model Context Protocol）サーバー**です。Claude Codeのサブプロセスとして起動し、外部サービスとセッションの橋渡しを行います。

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Telegram    │────▶│  MCP Server      │────▶│  Claude Code    │
│  (スマホ)    │◀────│  (Channel)       │◀────│  Session        │
└─────────────┘     └──────────────────┘     └─────────────────┘
     Bot API          サブプロセス              ローカル実行
```

外部からのメッセージは `<channel>` タグとしてClaude Codeのコンテキストに届き、Claudeはそれを読んで応答します。応答はChannelが提供するMCPツール（`reply`など）を通じて外部サービスに送り返されます。

### 利用可能なChannel

リサーチプレビュー段階では、以下の3つが公式に提供されています。

| Channel | 概要 |
|---------|------|
| **Telegram** | Bot APIベースのポーリング方式。スマホからDMで指示を出せる |
| **Discord** | DM・サーバー対応のBotチャンネル |
| **Fakechat** | ローカルホスト（`localhost:8787`）で動くデモUI。動作確認用 |

いずれも [Bun](https://bun.sh/) ランタイムが必要です。

### 従来の仕組みとの違い

:::message
Channelsは「プッシュ型」のイベント配信です。通常のMCPサーバーがClaude Codeからオンデマンドで呼び出される（プル型）のに対し、Channelsは外部からのメッセージを能動的にセッションに送り込みます。
:::

### 制限事項

- **リサーチプレビュー**: 仕様やプロトコルは今後変更される可能性がある
- **認証方式**: `claude.ai` ログインが必要（APIキー認証は非対応）
- **Team/Enterprise**: 管理者が `channelsEnabled` を有効にする必要がある
- **権限**: Telegram経由でもClaude Codeの通常の権限モデルが適用される。許可プロンプトが出た場合、不在時はセッションが一時停止する

## 本記事の構成

Telegram ChannelをZellij + Ghosttyと組み合わせて、リモートから開発指示を出せる環境を構築します。

**構成の全体像:**

```
スマホ (Telegram) → Telegram Bot API → Claude Code セッション (Zellij内)
```

:::message alert
**ターミナルを閉じる ＝ Claude Codeセッションが終了する ＝ Channelsが切断される**
外出中にスマホから指示を送る「非同期ワークフロー」を実現するには、ターミナルセッションをバックグラウンドで維持する必要があります。そこで使うのが **Zellij** です。
:::

**この構成のメリット:**

- PCの前にいなくてもClaude Codeに作業を依頼できる
- 外出先や家事の合間にスマホから進捗確認・追加指示が可能
- Zellijのデタッチにより、ターミナルを閉じてもセッションが維持される
- 帰宅後にZellijでアタッチし直せば、作業の続きをシームレスに確認できる

## Zellijとは

**Zellij**（ゼリージュ）は、Rust製のモダンなターミナルマルチプレクサです。tmuxと同じ「セッション永続化」ができるツールですが、大きな違いは**操作ガイドが画面に常時表示される**ことです。

- tmuxのようにキーバインドを暗記する必要がない
- 画面下部に「今何を押せばいいか」が表示される
- マウス操作にも対応
- デフォルト設定のままで十分使える

> tmuxで挫折した人のための、やさしいマルチプレクサ。

### GhosttyとZellijの役割

- **Ghostty** = ターミナルエミュレータ（画面を表示する側）
- **Zellij** = ターミナルマルチプレクサ（セッションを管理する側）

役割が別なので、**ターミナルエミュレータの中でZellijを起動する**のが正しい使い方です。ターミナルを閉じてもZellijのセッションはバックグラウンドで残ります。

:::message
**WezTermやiTerm2などでも同じです。** ターミナルエミュレータはどれを使ってもOK（Ghostty、WezTerm、iTerm2、Alacritty、標準Terminal.appなど）。タブやペイン機能があるターミナルでも、ウィンドウを閉じればプロセスは終了します。セッションを永続化できるのはZellijやtmuxのような**マルチプレクサだけ**です。
:::

## 前提環境

| ツール | 用途 | バージョン例 |
|--------|------|-------------|
| [Ghostty](https://ghostty.org/) | ターミナルエミュレータ | 1.x |
| [Zellij](https://zellij.dev/) | ターミナルマルチプレクサ | 0.41+ |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | AIコーディングアシスタント | v2.1.80以降 |
| [Bun](https://bun.sh/) | Telegram MCPサーバーのランタイム | 1.x |
| Telegram | モバイルメッセージングアプリ | - |

### Zellijのインストール

**macOS:**

```bash
brew install zellij
```

**Ubuntu / Debian:**

```bash
sudo apt install -y zellij
```

または公式バイナリ:

```bash
curl -L https://github.com/zellij-org/zellij/releases/latest/download/zellij-x86_64-unknown-linux-musl.tar.gz | tar xz
sudo mv zellij /usr/local/bin/
```

**インストール確認:**

```bash
zellij --version
# 例: zellij 0.41.2
```

## セットアップ手順

### 1. Telegram Botの作成

Telegram上で [@BotFather](https://t.me/BotFather) にDMを送り、`/newbot` コマンドでBotを作成します。

1. `/newbot` を送信
2. **Name**（表示名）を入力 — 任意の名前でOK
3. **Username**（`bot` で終わる一意のハンドル）を入力 — 例: `my_claude_assistant_bot`
4. BotFatherがトークンを返す — `123456789:AAHfiqksKZ8...` のような形式

このトークンを控えておきます。

### 2. Telegramプラグインのインストール

Claude Codeのセッション内で以下を実行します。

```
/plugin install telegram@claude-plugins-official
```

### 3. Botトークンの設定

```
/telegram:configure <BotFatherから取得したトークン>
```

トークンは `~/.claude/channels/telegram/.env` に `chmod 600` で保存されます。

### 4. Zellijセッションを作成してChannelsを起動

ここからがZellijとの連携です。Ghosttyを開いて以下を実行します。

```sh
# Zellijセッションを作成（既に存在すれば再接続）
zellij attach --create channels

# Zellijセッション内でClaude Codeを起動
claude --channels plugin:telegram@claude-plugins-official
```

`zellij attach --create channels` は「`channels`という名前のセッションがなければ作る、あれば繋ぐ」コマンドなので、毎回このコマンドでOKです。

:::message
通常の `claude` コマンドではTelegramサーバーは接続されません。`--channels` フラグが必須です。
:::

### 5. ペアリング

Claude Codeが起動した状態で、スマホのTelegramからBotにDMを送ります。Botが6文字のペアリングコードを返すので、Claude Codeセッション内で以下を実行します。

```
/telegram:access pair <6文字のコード>
```

これでペアリング完了です。次のDMからClaude Codeに届くようになります。

### 6. アクセスポリシーのロックダウン

ペアリング後、セキュリティのためにポリシーを `allowlist` に変更します。これにより、知らないユーザーがBotにDMしてもペアリングコードが返らなくなります。

```
/telegram:access policy allowlist
```

### 7. デタッチ（セッションから離脱）

`Ctrl+o` → `d`

画面下部のガイドにも表示されています。これでGhosttyを閉じてもClaude Codeは動き続けます。

### 8. 確認したいとき（再接続）

Ghosttyを開いて以下を実行します。

```sh
zellij attach channels
```

ログやステータスを確認できます。終わったら再び `Ctrl+o` → `d` でデタッチ。

## Zellij基本操作チートシート

| やりたいこと | 操作 | 補足 |
|-------------|------|------|
| セッション作成 & 接続 | `zellij attach --create channels` | 毎回これでOK |
| デタッチ（離脱） | `Ctrl+o` → `d` | セッションは残る |
| セッション再接続 | `zellij attach channels` | Ghostty再起動後でもOK |
| セッション一覧 | `zellij list-sessions` | 略称: `zellij ls` |
| セッション終了 | `zellij kill-session channels` | 完全にセッションを破棄 |
| ペイン分割（横） | `Ctrl+p` → `d` | 画面を上下に分割 |
| ペイン分割（縦） | `Ctrl+p` → `r` | 画面を左右に分割 |
| ペイン移動 | `Alt` + 矢印キー | ペイン間を移動 |
| 新しいタブ | `Ctrl+t` → `n` | タブを追加 |

> 覚えなくても大丈夫。画面下部に操作ガイドが常時表示されているので、それを見ればOK。

## スリープ・蓋閉じに関する重要な注意

:::message alert
**PCがスリープすると、Zellijセッションは残るが Claude Codeのプロセスは凍結されます。スリープ中はTelegramからの指示を処理できません。**
:::

### 状態別の動作

| PC状態 | Zellijセッション | Channels動作 |
|--------|-----------------|-------------|
| Ghosttyを閉じただけ | 生存 | 動作する |
| PCスリープ（蓋閉じ） | 残るが凍結 | 応答しない |
| PCスリープ → 復帰 | 復帰 | 復帰する |
| PC再起動 / シャットダウン | 消滅 | 再セットアップ必要 |

### 外出中もChannelsを使いたい場合

蓋を閉じてもスリープしない設定にする必要があります。

**macOS:**

1. **システム設定** → **バッテリー** → **オプション**
2. 「ディスプレイがオフのときに自動でスリープさせない」を有効化

または `caffeinate` コマンドで一時的にスリープを抑止:

```bash
caffeinate -s &
```

`-s` はAC電源接続時にスリープを防止します。外出前にこれを実行してから蓋を閉じます。

**Linux:**

```bash
# systemdの場合
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

または蓋閉じ時の動作を変更:

```
# /etc/systemd/logind.conf
HandleLidSwitch=ignore
```

```bash
sudo systemctl restart systemd-logind
```

## 実際の使い方

### スマホからの指示例

Telegramで以下のようなメッセージを送るだけでClaude Codeが動きます。

```
git statusを確認して
```

```
src/components/Header.tsxのレスポンシブ対応をお願い
```

```
さっきの変更をコミットして。メッセージは「feat: add responsive header」で
```

### ファイルの送信

スマホで撮ったスクリーンショットをTelegramで送ると、Claude Codeがその画像を読み取って対応できます。例えば、UIのバグのスクリーンショットを送って「この表示を直して」と依頼するようなワークフローが可能です。

:::message
Telegramは写真を圧縮します。高解像度が必要な場合は、長押し→「ファイルとして送信」を選択してください。
:::

## 推奨ワークフロー

### 毎日の運用

1. **朝**: Ghosttyを開く → `zellij attach --create channels` → `claude --channels plugin:telegram@claude-plugins-official` → `Ctrl+o` → `d` でデタッチ
2. **日中**: PCの蓋は開けたまま or スリープ無効にして放置。外出先からTelegramで指示
3. **帰宅後**: `zellij attach channels` で状況確認

### PC再起動後

```bash
zellij attach --create channels
claude --channels plugin:telegram@claude-plugins-official
# Ctrl+o → d
```

セッションは再起動で消えるので、再度作成します。

## セキュリティ上の注意点

Telegram Botは公開アドレス可能なため、以下の点に注意してください。

- **allowlistポリシー推奨**: ペアリング完了後は `allowlist` に切り替える
- **トークン管理**: Botトークンは `chmod 600` で保護されていますが、共有PCでは注意
- **操作権限**: Telegram経由でもClaude Codeの通常の権限モデルが適用される（ファイル編集やコマンド実行は許可設定に従う）
- **メッセージ履歴なし**: Telegram Bot APIにはメッセージ履歴・検索機能がないため、過去のやり取りはClaude Codeのコンテキスト内にのみ存在する

## トラブルシューティング

### Botが反応しない

- `--channels` フラグ付きでClaude Codeを起動しているか確認
- Zellijセッションが生きているか確認: `zellij list-sessions`
- PCがスリープしていないか確認
- ネットワーク接続が維持されているか確認

### ペアリングコードが返ってこない

- `dmPolicy` が `disabled` になっていないか確認: `/telegram:access`
- すでにallowlistに追加済みの場合はペアリング不要（そのままDMが届く）

### Zellijセッションが見つからない

```sh
# 既存セッション一覧を確認
zellij list-sessions

# セッションが残っていれば再アタッチ
zellij attach channels

# PC再起動後はセッションが消えているので再作成
zellij attach --create channels
```

## まとめ

Ghostty + Zellij + Claude Code + Telegramの組み合わせにより、PCの前にいなくてもAIペアプログラマーに作業を依頼できる環境が構築できます。Zellijのデタッチ機能がClaude Codeのセッション維持を担い、TelegramのChannelsプラグインがモバイルとの橋渡しをしてくれます。

:::message
**Zellij + スリープ無効 = Channelsの非同期ワークフローを実現する最小構成**
tmuxのようにキーバインドを暗記する必要がなく、画面に操作ガイドが表示されるので「覚えられない」問題が起きません。あとはスリープだけ気をつければOK。
:::

特に「ちょっとした修正を外出先から依頼したい」「家事の合間に進捗を確認したい」といったユースケースで威力を発揮します。セットアップも10分程度で完了するので、Claude Codeユーザーにはぜひ試してみてほしい構成です。
