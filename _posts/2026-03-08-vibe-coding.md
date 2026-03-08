---
layout: post
title: "Vibe Coding完全ガイド：AIに「雰囲気」で頼んでコードを書いてもらう新スタイル"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["vibecoding", "ai", "claudecode", "cursor", "llm"]
date: "2026-03-08 12:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-03-08-vibe-coding)でも公開しています。


:::message
この記事は **Claude Code**（claude.ai/code）を使用して作成しました。
:::

## はじめに

「コードを書く」という行為が大きく変わりつつあります。

2025年初頭、AI研究者の **Andrej Karpathy**（元OpenAI / Tesla）が提唱した **「Vibe Coding（バイブコーディング）」** という概念が、エンジニアコミュニティに衝撃を与えました。

> *"There's a new kind of coding I call 'vibe coding', where you fully give in to the vibes, embrace exponentials, and forget that the code even exists."*
> — Andrej Karpathy

直訳すると「雰囲気でコーディングする」。自分でコードを書くのではなく、**やりたいことを自然言語で伝えてAIに全部任せる**スタイルです。

本記事では、Vibe Codingの概念・各AIツールの具体的な使い方・向いているケース・注意点まで、わかりやすく解説します。

---

## Vibe Coding とは何か？

Vibe Codingは、以下の考え方を中心に据えたプログラミングスタイルです。

- コードの細かい実装は **AIが書く**
- 人間は **「何を作りたいか」** を自然言語で伝える
- エラーが出たら **そのままAIに貼り付けて直してもらう**
- コード自体を深く理解しなくてもプロダクトが作れる

これは「手を動かさない怠惰な開発」ではなく、**抽象度の高いレベルで人間がコントロールし、実装の詳細をAIに委譲する**という考え方です。

### 従来の開発スタイルとの比較

| 項目 | 従来のコーディング | Vibe Coding |
|------|------------------|-------------|
| 主な作業者 | 人間がコードを書く | AIがコードを書く |
| 人間の役割 | 実装・デバッグ | 要件定義・レビュー・方向修正 |
| 習得コスト | 高い（言語・フレームワーク知識が必要） | 低い（自然言語で指示できればOK） |
| 向いている場面 | 大規模・長期プロジェクト | プロトタイプ・個人開発・PoC |
| コードの品質管理 | 書いた本人が把握 | レビューが必要 |

---

## なぜ今、Vibe Coding が注目されるのか？

### LLMの急速な進化

GPT-4、Claude 3.5 Sonnet以降、コード生成の品質が劇的に向上しました。2026年現在では、複数ファイルにまたがる変更や、設計の提案・リファクタリングまでAIが担えるようになっています。

### AIコーディングツールの充実

| ツール | 種別 | 特徴 |
|--------|------|------|
| **Claude Code** | CLIエージェント | ターミナルベース・自律的なファイル操作・MCP対応 |
| **Cursor** | AIエディタ（VSCode派生） | インラインチャット・コードベース全体の理解 |
| **GitHub Copilot** | IDE拡張 | GitHub公式・Copilot Chat・PR要約機能 |
| **OpenCode** | CLIエージェント（OSS） | 75以上のAIプロバイダー対応・TUI操作 |
| **Windsurf** | AIエディタ | Cascade機能・コンテキスト自動収集 |

これらのツールが成熟し、「AIに任せる」ハードルがどんどん下がっています。

---

## Vibe Coding の基本ステップ

### ステップ1：やりたいことを日本語で書く

細かい技術仕様よりも、**「何を作りたいか」「どう動いてほしいか」** を自然に伝えます。

```
ユーザーがURLを入力すると、そのページのOGP画像・タイトル・説明文を
取得して表示するWebアプリを作ってください。
ReactとTailwind CSSを使い、シンプルなUIにしてください。
```

### ステップ2：AIの出力をそのまま動かしてみる

完璧を求めずに、まず動かしてみることが重要です。エラーが出たら…

### ステップ3：エラーはそのままAIに渡す

```
以下のエラーが出ました。直してください：

TypeError: Cannot read properties of undefined (reading 'title')
    at OgpCard (src/components/OgpCard.tsx:12:23)
```

コピペするだけでOK。自分でデバッグしようとしない、というのがVibe Codingの核心です。

### ステップ4：「こんな感じにしたい」でUIを調整

```
カードのデザインをもっとシャドウを強くして、
ホバー時にアニメーションを追加してください。
Notion風の雰囲気にしてほしいです。
```

「Notion風」「シンプルに」「ミニマル」といった抽象的な表現もAIは理解してくれます。

---

## 各AIツールの使い方

### 1. Claude Code — ターミナルで動く最強エージェント

Claude CodeはAnthropicが提供するCLIベースのAIエージェントです。**ファイルの作成・編集・コマンド実行・GitHubへのPR作成**まで、ターミナルから自律的にこなします。

#### 基本的な使い方

```bash
# プロジェクトディレクトリで起動
claude

# 指示を日本語で入力するだけ
> Next.jsでTodoアプリを作ってください。
  タスクの追加・削除・完了チェック機能が欲しいです。
  shadcn/uiを使ってきれいなデザインにしてください。
```

#### CLAUDE.md でプロジェクトの「雰囲気」を固定する

プロジェクトルートに `CLAUDE.md` を置くと、AIが毎回読み込む前提知識を設定できます。

```markdown
# プロジェクト概要
- Next.js 15 + TypeScript + Tailwind CSS
- コンポーネントはshadcn/uiを使用すること
- 状態管理はZustandを使用すること
- コミットメッセージは日本語で書くこと
```

毎回説明しなくても、AIがプロジェクトの方針を把握してくれます。

#### Claude Code をさらに使いこなすために

Claude Code には Vibe Coding をより強力にする機能が揃っています。

:::message
**関連記事**
- [Claude Code のSkillで自作スラッシュコマンドを作ろう](https://zenn.dev/long910/articles/2026-02-23-claude-code-skills) — 繰り返し使う指示をコマンド化して効率アップ
- [Claude Code のAgent Teamsで複数AIが協調するチームを作ろう](https://zenn.dev/long910/articles/2026-02-23-claude-code-agent-teams) — 複数のAIに並行して作業させる
- [Claude Code をスマホ・タブレットからリモート操作する](https://zenn.dev/long910/articles/2026-03-01-claude-code-remote-control) — 外出先からVibe Codingを継続
- [MCP（Model Context Protocol）入門](https://zenn.dev/long910/articles/2026-02-22-mcp-introduction) — SlackやGitHubとの連携でAIの行動範囲を広げる
- [Claude Code の使用量上限とうまく付き合う方法](https://zenn.dev/long910/articles/2026-02-21-claude-code-usage-limit) — 長時間Vibe Codingするときのコスト管理
:::

---

### 2. Cursor — エディタに溶け込むAI

Cursorは VS Code をベースにしたAI統合エディタです。既存のVS Code拡張がそのまま使えるため、移行コストが低いのが特徴です。

#### Composer（作曲家）機能でVibe Coding

`Cmd/Ctrl + I` でComposerを開くと、複数ファイルにまたがる変更を一度に指示できます。

```
# Composerへの指示例
ユーザー認証機能を追加してください。
- メール・パスワードでのログイン
- JWTトークンによるセッション管理
- ログアウト機能
既存のAPIの構造に合わせて実装してください。
```

#### Chat でコードベース全体に質問

`Cmd/Ctrl + L` でChatを開き、`@codebase` をつけると、プロジェクト全体を参照した回答が得られます。

```
@codebase この認証フローはどこで定義されていますか？
@codebase データベースのスキーマはどうなっていますか？
```

#### `.cursorrules` でルールを設定

```
# .cursorrules
- TypeScriptの型を必ず明示すること
- コメントは日本語で書くこと
- コンポーネントはshadcn/uiを使うこと
- テストはVitestで書くこと
```

---

### 3. GitHub Copilot — GitHubと密接に統合されたAI

GitHub Copilotは、VS Code・JetBrains・Neovimなど多くのエディタで使えます。GitHubのリポジトリとの親和性が高く、**PRの要約・Issue対応・コードレビュー**までカバーします。

#### インラインでの補完

コードを書き始めると自動でサジェストが出ます。`Tab` で確定するだけ。

```typescript
// 「ユーザーのメールアドレスを検証する関数」とコメントを書くと…
// → Copilotが実装を自動補完してくれる
function validateEmail(email: string): boolean {
  // ↑ここまで書いたらCopilotが続きを提案
```

#### Copilot Chat でVibe Coding

`Cmd/Ctrl + Shift + I` でCopilot Chatを開き、自然言語で指示します。

```
/fix このコードのバグを直して
/explain このコードを日本語で説明して
/tests このコードのテストを書いて
/doc JSDocコメントを追加して
```

---

### 4. OpenCode — OSS・どのLLMでも使えるCLIエージェント

OpenCodeはSST（Serverless Stack）チームが開発したオープンソースのCLIエージェントです。75以上のAIプロバイダーに対応しており、**Claude・OpenAI・Gemini・ローカルLLMを自由に切り替えられる**のが強みです。

#### 基本的な使い方

```bash
# インストール
npm install -g opencode

# 起動（TUIが開く）
opencode

# モデルを指定して起動
opencode --model anthropic/claude-sonnet-4-5
```

#### Claude Code との使い分け

| 比較項目 | Claude Code | OpenCode |
|---------|-------------|----------|
| 対応モデル | Claude のみ | 75以上のプロバイダー |
| ライセンス | プロプライエタリ | MIT（OSS） |
| UI | シンプルなCLI | リッチなTUI |
| MCP対応 | あり | あり |

:::message
**関連記事**
OpenCode の詳細なインストール方法・設定・Claude Codeとの比較は以下の記事で詳しく解説しています。

→ [OpenCode完全ガイド：ターミナルで動くOSSのAIコーディングエージェント](https://zenn.dev/long910/articles/2026-03-04-opencode)
:::

---

### 5. Windsurf — コンテキストを自動収集するAIエディタ

WindsurfはCodeium社製のAI統合エディタです。**Cascade（カスケード）** という機能が特徴で、AIが自律的にコードベースを探索しながら作業を進めます。

#### Cascade での指示

```
新しいユーザープロフィールページを作成してください。
既存のデザインシステムに合わせて、
アバター・名前・自己紹介・フォロワー数を表示するレイアウトにしてください。
```

CascadeはAI自身がどのファイルを参照すべきかを判断し、必要な箇所を自動で調べながら実装します。

---

## ツール選定の目安

```
ターミナル派・自律作業重視 → Claude Code
VS Code ユーザー・移行コスト低め → Cursor
GitHub との連携重視 → GitHub Copilot
OSS・複数LLM使い分けたい → OpenCode
コンテキスト自動収集が欲しい → Windsurf
```

---

## Vibe Coding が向いているケース・向いていないケース

### 向いているケース ✅

- **個人開発・副業プロジェクト**：スピードが命のプロトタイプ
- **PoC（概念実証）**：アイデアを素早く形にしたい
- **非エンジニアのツール作成**：社内ツールやスクリプトを自分で作りたい人
- **不慣れな言語での開発**：普段使わない言語でのタスク
- **UI/UXのデザインと実装**：デザインの試行錯誤を素早く行いたい

### 向いていないケース ⚠️

- **大規模チーム開発**：コードレビューや品質管理が困難になる
- **セキュリティ要件が高いシステム**：AIの出力をそのまま使うのは危険
- **パフォーマンスが重要な部分**：AIが最適化を怠ることがある
- **長期メンテナンスが必要なプロダクト**：技術的負債が蓄積しやすい

---

## Vibe Coding の落とし穴と対策

### 落とし穴1：コードブラックボックス化

AIが書いたコードを理解せずに使い続けると、バグが出たときに手も足も出なくなります。

**対策**：重要な部分はコードレビューを習慣化し、「なぜこう書くか」をAIに説明させる。

```
このコードの動作を、初心者にもわかるように日本語で説明してください。
```

### 落とし穴2：セキュリティリスク

認証・認可・SQLクエリなど、セキュリティに関わる部分を無検証で使うのは危険です。

**対策**：セキュリティチェックリストを作り、AIに「このコードにセキュリティ上の問題はないか」と確認させる。

### 落とし穴3：技術的負債の蓄積

Vibe Codingで爆速で作ったコードは、一貫性やアーキテクチャが崩れがちです。

**対策**：定期的に「このコードをリファクタリングしてください」とAIに依頼する。また、最初に `CLAUDE.md` や `.cursorrules` でアーキテクチャ方針を定義しておく。

### 落とし穴4：コスト管理の失敗

長時間Vibe Codingをしていると、AIの使用量・コストが想定以上に膨らむことがあります。

:::message
**関連記事**
Claude Code のトークン消費・コスト管理については以下の記事が参考になります。

→ [Claude Code の使用量上限とうまく付き合う方法](https://zenn.dev/long910/articles/2026-02-21-claude-code-usage-limit)
:::

---

## Vibe Coding の未来

Vibe Codingは「プログラマーが不要になる」話ではありません。

むしろ、**プログラマーの役割が変わる**という話です。

- **コードを書く** → **コードを設計・レビューする**
- **実装に詳しい人** → **何を作るべきかを判断できる人**
- **タイピングが速い人** → **AIに的確に指示できる人**

AIが「実装者」になり、人間が「アーキテクト・プロデューサー」になる世界。その入口として、Vibe Codingは非常に実践的なアプローチです。

---

## まとめ

| ポイント | 内容 |
|---------|------|
| Vibe Codingとは | 自然言語でAIに指示してコードを書いてもらうスタイル |
| 提唱者 | Andrej Karpathy（2025年） |
| 向いている場面 | プロトタイプ・個人開発・PoC |
| おすすめツール | Claude Code、Cursor、GitHub Copilot、OpenCode、Windsurf |
| 注意点 | コードの理解・セキュリティ・技術的負債・コスト |

「コードが書けないから」と諦めていた人も、「効率を上げたい」エンジニアも、一度Vibe Codingを試してみてください。AIと一緒に作るプログラミングは、想像以上に楽しいものです。

---

## 参考リンク

- [Andrej Karpathy の元ポスト（X）](https://x.com/karpathy/status/1886192184808149217)
- [Claude Code 公式ドキュメント](https://docs.anthropic.com/en/docs/claude-code)
- [Cursor 公式サイト](https://www.cursor.com/)
- [OpenCode 公式サイト](https://opencode.ai/)
- [GitHub Copilot 公式サイト](https://github.com/features/copilot)
- [Windsurf 公式サイト](https://codeium.com/windsurf)
