---
layout: post
title: "ClaudeとGeminiの「Nano Banana」を連携して画像生成を自動化する完全ガイド"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["claude", "gemini", "画像生成", "mcp", "claudecode"]
date: "2026-02-28 18:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-02-28-claude-gemini-nano-banana)でも公開しています。


Claude Code を使っていると、「画像も一緒に生成できたら…」と思ったことはありませんか？

Google が開発した画像生成モデル **Nano Banana**（Gemini Flash Image）を Claude と組み合わせることで、テキスト生成と画像生成を自然な会話の流れでシームレスに行えるようになります。この記事では、連携の仕組みから実践的な使い方まで、丁寧に解説します。

---

## Nano Banana とは？

**Nano Banana** は、Google DeepMind が開発した画像生成・編集モデルです。正式名称は **Gemini Flash Image**（最新版は Gemini 3.1 Flash Image Preview、コードネーム: Nano Banana 2）で、2025年8月に初版がリリースされ、2026年2月に第2世代が公開されました。

### 主な特徴

| 機能 | 詳細 |
| ---- | ---- |
| テキストから画像生成 | 自然言語のプロンプトで高品質な画像を生成 |
| 画像編集 | 既存の画像をプロンプトで自由に修正 |
| i18n テキスト描画 | 多言語テキストを画像内に正確に描画 |
| アスペクト比制御 | 任意のアスペクト比で出力可能 |
| 最大4Kアップスケール | 高解像度への拡大処理 |
| キャラクター一貫性 | 最大5キャラクター・14オブジェクトの一貫性を維持 |
| SynthID 透かし | 全生成画像に透かし情報を埋め込み |

### 料金

2026年2月時点の API 料金は **$67 / 1,000枚**（Gemini Flash Image の場合）で、競合モデルと比較して低価格に設定されています。

:::message
Google AI Studio でアカウントを作成すれば、無料枠の範囲内でお試し利用が可能です。
:::

---

## Claude と Nano Banana を組み合わせる理由

Claude は優れた文章生成・コード生成 AI ですが、単体では画像生成はできません。一方、Nano Banana は画像生成は得意ですが、複雑な文脈管理や反復的な改善指示には限界があります。

**両者を組み合わせると：**

- Claude が意図を理解し、最適なプロンプトを生成 → Nano Banana が画像を出力
- Claude が生成結果を評価し、問題点を特定 → 自律的に再生成・修正
- 長いセッションでも Claude がコンテキストを保持 → 一貫したワークフローを維持

実際に開発者が試したところ、アプリアイコンを 100 枚以上反復生成するプロジェクトで $45 程度のコストで完成できた事例もあります。

---

## 連携方法 1: MCP サーバー経由（推奨）

**MCP（Model Context Protocol）** を使う方法が最も簡単で、Claude Desktop・Claude Code・Cursor など多くのクライアントで利用できます。

### 事前準備

1. **Gemini API キーの取得**
   [Google AI Studio](https://aistudio.google.com/) にアクセスし、「Get API key」からキーを発行します。

2. **Node.js のインストール**
   バージョン 18.0.0 以上が必要です。

### インストール

```bash
# 試しに使ってみる場合（インストール不要）
npx nano-banana-mcp

# グローバルインストール
npm install -g nano-banana-mcp
```

### Claude の設定ファイルに追記

#### Claude Desktop の場合

`~/.config/claude/claude_desktop_config.json`（Linux/Mac）または `%APPDATA%\claude\claude_desktop_config.json`（Windows）に以下を追加します。

```json
{
  "mcpServers": {
    "nano-banana": {
      "command": "npx",
      "args": ["nano-banana-mcp"],
      "env": {
        "GEMINI_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

#### Claude Code の場合

プロジェクトルートの `.mcp.json` に追記するか、グローバル設定に追加します。

```bash
# プロジェクトに追加
claude mcp add nano-banana npx nano-banana-mcp -e GEMINI_API_KEY=your-api-key-here

# グローバルに追加
claude mcp add --global nano-banana npx nano-banana-mcp -e GEMINI_API_KEY=your-api-key-here
```

### 設定の確認

```bash
claude mcp list
```

`nano-banana` が表示されれば設定完了です。

### MCP サーバーが提供するツール

| ツール名 | 機能 |
| -------- | ---- |
| `generate_image` | テキストプロンプトから画像を生成 |
| `edit_image` | 既存の画像をプロンプトで編集 |
| `continue_editing` | 直前に生成・編集した画像を継続して修正 |
| `get_last_image_info` | 現在の画像のメタデータを取得 |
| `configure_gemini_token` | API キーを動的に設定 |
| `get_configuration_status` | 設定状態を確認 |

### 使ってみる

Claude に話しかけるだけで画像生成が始まります。

```
ブログのヘッダー画像を作って。テーマは「AIと人間の共創」で、
青と白を基調としたシンプルなデザインにしてください。
横長（16:9）で出力してください。
```

Claude が `generate_image` ツールを呼び出し、生成された画像が会話に表示されます。

---

## 連携方法 2: Claude Code スキル経由

Claude Code には**スキル**という仕組みがあり、スラッシュコマンドで特定の処理を呼び出せます。Nano Banana 専用スキルを使うと、Gemini CLI の拡張機能として画像生成を呼び出せます。

### 事前準備

#### 1. Gemini CLI をインストール

```bash
npm install -g @google/gemini-cli
```

#### 2. API キーを設定

```bash
export NANOBANANA_GEMINI_API_KEY="your-api-key-here"
```

永続化するには `~/.zshrc` や `~/.bashrc` に追記します。

#### 3. Nano Banana 拡張機能をインストール

```bash
gemini extensions install https://github.com/gemini-cli-extensions/nanobanana
```

#### 4. Claude Code スキルをセットアップ

```bash
# スキルディレクトリを作成
mkdir -p ~/.claude/skills/nano-banana

# スキルファイルをダウンロード
git clone https://github.com/kkoppenhaver/cc-nano-banana /tmp/cc-nano-banana
cp -r /tmp/cc-nano-banana/* ~/.claude/skills/nano-banana/
```

### 使い方

スキルが認識されると、Claude Code 内でスラッシュコマンドが使えるようになります。

| コマンド | 用途 |
| -------- | ---- |
| `/generate` | テキストから画像を生成 |
| `/edit` | 既存の画像を編集 |
| `/restore` | 画像を修復・補完 |
| `/icon` | アプリアイコンを生成 |
| `/diagram` | フローチャートや図を生成 |
| `/pattern` | テクスチャやパターンを生成 |
| `/story` | 連続した画像（ストーリーボード）を生成 |

```
/icon アプリアイコンを作って。生産性向上アプリ用で、
緑と白のカラーで、ミニマルなデザインにしてください。
バリエーション3種類出してください。
```

---

## 実践的なユースケース

### ユースケース 1: アプリアイコンの反復生成

モバイルアプリのアイコン開発に非常に有効です。Claude が生成結果を分析し、デザインの問題点を特定して自動的に修正指示を出します。

```
【Claude へのプロンプト例】
ヘルスケアアプリのアイコンを作成してください。

要件：
- 心拍数・健康をイメージさせる
- 青と緑のグラデーション
- 最小限の要素でシンプルに
- iOS App Store のガイドラインに準拠

生成したら、デザインの改善点を分析して3回まで自動的に改善してください。
```

Claude は画像を生成するたびにフィードバックを行い、指定した回数まで自律的に品質を高めていきます。

### ユースケース 2: ブログ・SNS 向けサムネイル一括作成

記事のリストを渡すと、それぞれに適したサムネイルを一括で作成できます。

```
以下の記事タイトル5本分のサムネイル画像を作成してください。
各画像は1200×630px（OGP推奨サイズ）で、統一されたスタイルを使ってください。

1. AIで変わる開発者のワークフロー
2. TypeScriptベストプラクティス2026
3. Dockerコンテナのセキュリティ強化
4. チームで使うコードレビュー文化
5. パフォーマンス計測の自動化入門
```

### ユースケース 3: 多言語スクリーンショットの翻訳

Nano Banana はテキスト描画に強いため、アプリのスクリーンショット内の文字を別の言語に置き換えることができます。

```
このアプリのスクリーンショット（英語）を、
テキスト部分だけ日本語に翻訳した版を作成してください。
レイアウトやデザインはそのまま維持してください。
```

### ユースケース 4: 技術ドキュメント用の図解作成

コードの説明や仕組みを図解するのにも使えます。

```
以下のアーキテクチャを図解してください：

クライアント → API Gateway → Lambda × 3（認証/データ/通知）→ DynamoDB

矢印と各コンポーネントのラベルを含めて、
白背景でシンプルかつ技術的なデザインにしてください。
```

---

## 活用テクニック

### テクニック 1: プロンプトテンプレートを用意する

繰り返し使うスタイルやフォーマットをテンプレート化しておくと効率的です。

```markdown
## 画像生成テンプレート

### ブログサムネイル
- サイズ: 1200×630px
- スタイル: モダン、フラットデザイン
- カラー: メインカラー [色指定] + 白背景
- フォント: タイトル大きく、サブタイトル小さく
- 右上にロゴスペースを確保
```

Claude にこのテンプレートを渡してから生成指示を出すと、一貫したスタイルで出力できます。

### テクニック 2: 段階的な精製（イテレーション）

一発で完璧な画像を求めるより、段階的に精製する方が質の高い結果を得やすいです。

```
Step 1: まずラフな構成を生成してください
Step 2: 構成が決まったら色調を調整してください
Step 3: 細部のディテールを追加してください
Step 4: 最終チェックとして解像度を上げてください
```

### テクニック 3: 参照画像を活用する

編集機能を使うときは、参照する元画像を用意して渡すと精度が上がります。

```
この画像（参照）のスタイルを参考に、
別の製品（スマートフォン）の商品画像を作成してください。
背景のぼかし具合と光の当たり方を合わせてください。
```

### テクニック 4: MCP と Skill の使い分け

| 状況 | 推奨方法 |
| ---- | -------- |
| Claude Desktop で手軽に使いたい | MCP サーバー |
| Claude Code でコーディングと並行して使いたい | MCP サーバー or スキル |
| チームで設定を統一したい | MCP サーバー（`.mcp.json` で共有） |
| Gemini CLI を既に使っている | スキル経由 |
| オフライン・プロキシ環境で使いたい | スキル経由（柔軟性が高い） |

---

## トラブルシューティング

### API キーが認識されない

```bash
# 環境変数が正しく設定されているか確認
echo $GEMINI_API_KEY

# Claude Code の場合は MCP の設定を確認
claude mcp list
```

### 画像が生成されない

Google AI Studio の [API キー管理画面](https://aistudio.google.com/apikey) で以下を確認してください。

- API キーが有効であること
- 使用量の上限に達していないこと
- 対象の API（Gemini API）が有効になっていること

### 生成された画像の品質が低い

プロンプトに以下の要素を加えると改善することが多いです。

- **解像度**: `高解像度`、`4K`、`鮮明な`
- **スタイル**: `プロフェッショナル`、`フォトリアル`、`イラスト調`
- **構図**: `中央寄り`、`余白あり`、`ルールオブサーズ`

---

## セキュリティと注意点

:::message alert
**API キーの取り扱いに注意してください**

- API キーをコードやコミットに直接書かないこと
- 環境変数または `.env` ファイルで管理し、`.gitignore` に追加すること
- チームで共有する場合は、個人ごとに API キーを発行すること
:::

### 生成画像の著作権

Nano Banana（Gemini Flash Image）で生成された画像は基本的に利用者に帰属しますが、商用利用の際は Google の [利用規約](https://policies.google.com/terms) を必ず確認してください。また、SynthID 透かしが埋め込まれているため、AI 生成画像であることを識別できます。

### コスト管理

大量の画像を生成するプロジェクトでは、コストが予想以上に膨らむことがあります。

```bash
# 使用量を Google AI Studio で定期的に確認する
# https://aistudio.google.com/usage
```

必要に応じて Google Cloud の予算アラートを設定することをお勧めします。

---

## まとめ

Claude と Nano Banana（Gemini Flash Image）の連携を整理します。

**2つの連携方法：**

| 方法 | 難易度 | 主な用途 |
| ---- | ------ | -------- |
| MCP サーバー | 低（設定ファイルを追記するだけ） | Claude Desktop・Claude Code との自然な会話で画像生成 |
| Claude Code スキル | 中（Gemini CLI のセットアップが必要） | Claude Code 内でのコマンド形式の画像生成 |

**主なユースケース：**

- アプリアイコン・UI デザインの反復生成
- ブログやSNSのサムネイル作成
- 技術ドキュメント用の図解・フローチャート
- 多言語スクリーンショットの翻訳

テキスト AI と画像 AI を組み合わせることで、これまで別々のツールで行っていた作業を Claude との会話だけで完結できるようになります。ぜひ試してみてください。

---

## 参考リンク

- [nano-banana-mcp（GitHub）](https://github.com/ConechoAI/Nano-Banana-MCP) — MCPサーバーの実装
- [cc-nano-banana（GitHub）](https://github.com/kkoppenhaver/cc-nano-banana) — Claude Codeスキルの実装
- [YCSE/nanobanana-mcp（GitHub）](https://github.com/YCSE/nanobanana-mcp) — Gemini Vision & Image Generation MCP
- [Gemini 3 Pro Image（DeepMind）](https://deepmind.google/models/gemini-image/pro/) — Nano Banana Pro の公式ページ
- [Nano Banana + Claude Code スキルの活用事例](https://mkdev.me/posts/unlimited-image-generation-with-nano-banana-pro-and-custom-claude-code-skill) — 実際の開発事例
- [Gemini CLI 入門](https://zenn.dev/long910/articles/2025-06-30-gemini-cli-introduction) — Gemini CLI の基礎解説
