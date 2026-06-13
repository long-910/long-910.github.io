---
layout: post
title: "Claude Code で使える MCP サーバーを試してみた——実用的だったものを整理する"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["mcp", "claudecode", "ai", "claude", "productivity"]
date: "2026-06-13 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-13-mcp-servers-practical)でも公開しています。


## はじめに

MCP（Model Context Protocol）という単語を最近よく見かけるようになりました。Anthropic が 2024年末にオープンソースとして公開した規格で、「AI モデルと外部ツールをつなぐ USB みたいなもの」という説明をよく目にします。

気になって Claude Code に実際に入れて試してみたところ、思っていたより使えるものが多かったので整理しておきます。

---

## MCP とは何か

MCP（Model Context Protocol）は AI モデルとデータソース・ツールを接続するための標準プロトコルです。

```
Claude Code（クライアント）
    ↕ MCP
外部ツール（GitHub / ファイルシステム / ブラウザ / DB など）
```

公開から約1年半で、月間 SDK ダウンロード数は 9,700 万回を超え、GitHub スターは 86,000 以上に達しました。2026年4月には Linux Foundation 傘下の Agentic AI Foundation（AAIF）に寄贈され、AWS が 54本、Google Cloud も公式サーバーをリリースするなど、特定ベンダーの仕様にとどまらない広がりを見せています。

---

## 実際に試してよかった MCP サーバー

### 1. GitHub MCP（GitHub 公式）

**インストール**

```bash
# GitHub 公式サーバー（Go 製バイナリ、Docker 経由）
docker pull ghcr.io/github/github-mcp-server
claude mcp add github -- docker run -i --rm \
  -e GITHUB_PERSONAL_ACCESS_TOKEN \
  ghcr.io/github/github-mcp-server
```

:::message
以前は `@modelcontextprotocol/server-github`（npm）が広く使われていましたが、現在は archived 扱いになり、GitHub が Go で書き直した [`github/github-mcp-server`](https://github.com/github/github-mcp-server) が公式推奨です。環境変数名も `GITHUB_TOKEN` ではなく `GITHUB_PERSONAL_ACCESS_TOKEN` です。ローカルインストール不要のリモート版も提供されています。
:::

**何ができるか**

| 操作 | 例 |
|------|------|
| PR の確認 | 「open な PR を一覧にして」 |
| Issue 管理 | 「bug ラベルの Issue を優先度順に整理して」 |
| コード検索 | 「リポジトリ内で deprecated な関数を探して」 |
| コメント投稿 | 「このレビューコメントをそのまま PR に投稿して」 |

ブラウザを開かずに Issue の確認や PR のレビューコメント投稿ができるので、記事を書きながら別タブを行き来する手間が減りました。

---

### 2. Filesystem MCP（公式）

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /Users/your_name
```

ディレクトリ単位でアクセスを許可する設計なので、必要な範囲だけ渡せます。

**効いた場面**

- プロジェクト横断で同じパターンを探して一括修正
- 「このフォルダ内のすべての markdown ファイルのタイトルを一覧にして」
- ローカルにある複数のファイルを比較しながら整理

Claude Code 単体でも Read/Edit ツールはありますが、MCP 経由だと「どのファイルを見るか」を自分で指定しなくてもよくなる場面があります。

---

### 3. Brave Search MCP

```bash
claude mcp add brave-search -- npx -y @modelcontextprotocol/server-brave-search
# BRAVE_API_KEY 環境変数が必要（無料プランあり）
```

Claude の知識は学習カットオフ以前の情報に限られます。Brave Search MCP を入れると、Claude から直接ウェブ検索できるようになります。

```
「Context7 の最新リリースノートを調べて要約して」
→ Claude が Brave Search を呼び出し → 結果を要約
```

記事を書くときに最新情報を調べながら文章を作る、というフローがスムーズになりました。

---

### 4. Playwright MCP（Microsoft 公式）

```bash
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

Claude がブラウザを操作できるようになります。

**できること**

- 「このページのスクリーンショットを撮って」
- 「ログインして、このフォームに入力して送信して」
- 「ステージング環境の dashboard にアクセスして表示を確認して」

API を持たないウェブサービスの操作を自動化できるのが強みです。ブラウザのテスト自動化にも使えます。ただし、ページの構造次第で失敗することもあるので、重要な操作は結果を確認しながら進めるのが無難です。

---

### 5. Context7 MCP

```bash
claude mcp add context7 -- npx -y @upstash/context7-mcp
```

ライブラリの最新ドキュメントを取得して、Claude に渡してくれるサーバーです。

```
「use context7」と書いてから質問すると
→ 該当ライブラリの最新ドキュメントを取得して回答に反映
```

学習データが古い Claude に最新の API の使い方を教えた上で回答させる、という仕組みです。「この関数は存在しない」と言われたとき、Context7 経由で最新仕様を参照させると正しく回答できることがありました。

---

### 6. Memory MCP（Mem0）

```bash
claude mcp add mem0 -- npx -y @mem0/mcp-server
# MEM0_API_KEY が必要
```

会話をまたいで情報を記憶しておけるサーバーです。

**使えそうな場面**

- 「このプロジェクトの方針」「よく使うコマンド」を登録しておく
- セッションが変わっても前回の文脈を引き継ぐ
- 複数のプロジェクトにまたがる共通情報を管理する

Claude Code 自体にも `/memory` 機能がありますが、Mem0 はプラットフォーム横断（Claude.ai デスクトップ・API など）で共有できる点が違います。

---

### 7. Sequential Thinking MCP

```bash
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

ステップを逐次的に記録しながら推論させるサーバーです。複雑な設計タスクや、判断基準が多い問題に使うと整理しやすくなります。

「どのアーキテクチャを選ぶべきか」といった問いを投げると、考えの過程を記録しながら結論を出してくれます。結果より**過程が見えるようになる**点が、検証に便利でした。

---

### 8. Google Workspace 系（Gmail / Calendar / Drive）

```bash
# Claude.ai の設定画面から OAuth 経由で接続
```

Claude Code のマーケットプレイスから追加できます。

| サーバー | できること |
|---------|-----------|
| Gmail | メールの検索・要約・下書き作成 |
| Google Calendar | スケジュール確認・イベント追加 |
| Google Drive | ファイル検索・内容の確認 |

「今週のミーティング一覧を出して」「この件名のメールを要約して」といった使い方ができます。ただし、送信・削除などの操作は確認を挟むようにしておくと安心です。

---

## 組み合わせの例

単体より組み合わせると効果が出やすかったパターンです。

```
【記事執筆フロー】
Brave Search（最新情報を調べる）
+ Filesystem（ローカルのドラフトを読む・書く）
+ GitHub（参照リンクを確認する）
```

```
【コードレビューフロー】
GitHub（PR の差分を取得）
+ Context7（使われているライブラリのドキュメント参照）
+ Sequential Thinking（複雑な変更の影響範囲を整理）
```

---

## 実際に使ってみての注意点

### 同時に入れすぎない

Cursor のドキュメントでは、MCP サーバーが提供するツール数の合計が約 40 を超えると挙動が不安定になるとされています。Claude Code でも大量のサーバーを追加すると、どのツールを使うか選択が遅くなる場合があります。まず 3〜5 本に絞るのが現実的です。

### 認証情報の管理に注意

API キーや OAuth トークンを扱うサーバーが多いです。`.env` ファイルやシークレット管理ツールを使い、キーをそのままコード内に書かないようにします。

### 動作確認は小さく始める

ブラウザ操作・メール送信など副作用のある操作は、最初は読み取りだけで試してから範囲を広げると安全です。

---

## 現状の把握

MCP サーバーは 2026年6月時点で、Glama 単体で2万本超、複数のレジストリ（Smithery・mcp.so 等）を合算すると5万本規模に達しています。全部確認することは現実的ではないので、自分のワークフローで何がボトルネックになっているかを先に整理してから選ぶのが効率的だと感じました。

今回試した中では、**GitHub + Brave Search + Context7** の組み合わせが、コードを書きながら調べる作業に一番フィットしました。

---

## まとめ

| MCP サーバー | 特に効いた場面 |
|------------|--------------|
| GitHub | PR / Issue 管理をブラウザなしで |
| Filesystem | ファイル横断の検索・編集 |
| Brave Search | 最新情報のリアルタイム調査 |
| Playwright | ブラウザ操作の自動化 |
| Context7 | ライブラリの最新ドキュメント参照 |
| Memory（Mem0） | セッションをまたいだ文脈の保持 |
| Sequential Thinking | 複雑な問題の推論過程の可視化 |
| Google Workspace | Gmail / Calendar / Drive の連携 |

MCP は「AI に何かをやらせる」というより「AI が参照・操作できる範囲を広げる」ものだと理解すると、何を入れるべきかが見えやすくなりました。まず自分の作業でよく繰り返していることを思い浮かべて、そこに合うサーバーを 1〜2 本試してみるのがよさそうです。

---

## 参考リンク

- [Model Context Protocol — 公式サイト](https://modelcontextprotocol.io/)
- [Anthropic — MCP 発表](https://www.anthropic.com/news/model-context-protocol)
- [modelcontextprotocol/servers — GitHub](https://github.com/modelcontextprotocol/servers)
- [github/github-mcp-server — GitHub 公式 MCP サーバー](https://github.com/github/github-mcp-server)
- [Best MCP Servers May 2026 — andrew.ooo](https://andrew.ooo/answers/best-mcp-servers-may-2026/)
- [The 15 MCP Servers Worth Wiring Into Claude Code and Cursor (2026) — Codersera](https://codersera.com/blog/best-mcp-servers-claude-code-cursor-2026/)
