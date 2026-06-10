---
layout: post
title: "Claude Fable 5 登場——Mythosクラス初の一般公開、6/22までサブスクで無料"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["Claude", "Anthropic", "AI", "LLM", "生成AI"]
date: "2026-06-10 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-10-claude-fable-5-guide)でも公開しています。


2026年6月9日、Anthropicが **Claude Fable 5** を一般公開しました。これまで研究パートナー限定だった「Mythosクラス」が、ついに誰でも使える形で登場します。

:::message
**サブスク（Pro / Max / Team / Enterprise）ユーザーは 6月22日まで追加料金なし** で Claude Fable 5 が使えます。
6月23日以降は容量が安定するまで Usage Credits（API課金）に移行予定。
:::

---

## 🔮 Fable 5 と Mythos 5 ——同じ魂、違う鎧

```
┌─────────────────────────────────────────────────────────┐
│              Anthropic モデル体系（2026年6月）           │
├──────────────────────┬──────────────────────────────────┤
│  Claude Mythos 5     │  審査済みパートナー限定           │
│                      │  安全クラシファイア：なし          │
│                      │  フル能力を解放                   │
├──────────────────────┼──────────────────────────────────┤
│  Claude Fable 5 ✨   │  一般公開（本記事の対象）         │
│                      │  安全クラシファイア：あり          │
│                      │  95%+ のセッションはFable単独完結 │
└──────────────────────┴──────────────────────────────────┘
```

Fable 5 は Mythos 5 と**同一の基盤モデル**です。違いは高リスク領域（サイバーセキュリティ・生化学・蒸留）を検知した際にOpus 4.8へ自動ルーティングするクラシファイアの有無だけ。切り替えが起きるのはセッション全体の **5%未満** です。

---

## 📊 Opus 4.8 との比較

### ベンチマーク

| 指標 | 🧚 Fable 5 | Opus 4.8 |
|---|:---:|:---:|
| SWE-Bench Pro | **80.3%** | 69.2% |
| FrontierCode | **約2倍** | ベースライン |
| 長時間・複雑タスク | ◎ 大きくリード | △ 途中で失速 |
| 短期・明確なタスク | ○ | ○ ほぼ同等 |

> タスクが長く複雑であるほど Fable 5 のリードは広がります。

### レイテンシ

| モデル | 典型的な応答時間（コーディング） |
|---|---|
| Opus 4.8 | ⚡ 3〜15 秒 |
| Fable 5 | 🐢 60秒〜数分 |

### データ保持

| モデル | 保持ポリシー |
|---|---|
| Opus 4.8 | ゼロデータ保持オプション **あり** |
| Fable 5 | 安全クラシファイアのため **30日間保持必須** |

:::message alert
機密データを扱う業務では、Fable 5 のデータ保持ポリシーに注意してください。
:::

---

## ✨ Fable 5 が得意なこと

### 1️⃣ プロンプト一発でビデオゲームを生成

ゲームのコンセプトを数文で書くだけで、**3Dレンダリング付きのブラウザプレイアブルなゲーム**が生成されます。

- Babel図書館エクスプローラー
- 自己認識型 Snake
- Factorio・ポケモンFireRed を自律プレイ

### 2️⃣ 長時間・大規模な自律エージェントタスク

AI研究者 Ethan Mollick 氏のレポート：
> 「数十時間にわたって数十ページの仕様書を実行し続けた。これまで使ってきたどの公開モデルをも大きく上回る。」

### 3️⃣ ソフトウェアエンジニアリング

GitHub Copilot にも同日統合。SWE-Bench Pro スコア **80.3%** はコーディング特化モデルを含めても最高水準です。

---

## 💰 料金体系

### API 従量課金

```
┌──────────────────┬──────────────┬───────────────┐
│ モデル           │ 入力 /1M     │ 出力 /1M      │
├──────────────────┼──────────────┼───────────────┤
│ 🧚 Fable 5      │   $10        │    $50        │
│ Opus 4.8         │    $5        │    $25        │
│ Sonnet 4.6       │    $3        │    $15        │
│ Haiku 4.5        │   $0.80      │     $4        │
└──────────────────┴──────────────┴───────────────┘
```

Fable 5 は Opus 4.8 の **約 2 倍** の料金です。

### サブスクリプション（6/9〜6/22）

:::message
**Pro / Max / Team / Enterprise（座席制）に加入中のユーザーは 6月22日まで無料で使えます。**

⚠️ ただし Fable 5 の使用量は Opus 4.8 の **約 2 倍** としてクォータにカウントされます。同じ作業量でも月次上限の消費ペースが倍になる点に注意。
:::

### 6月23日以降

```
6月22日まで ─────▶ サブスク内で無料（クォータ2倍消費）
6月23日以降 ─────▶ Usage Credits（APIレートで課金）
容量確保後  ─────▶ 標準プラン機能として復帰予定（日程未定）
```

---

## 🌐 利用可能なプラットフォーム

| プラットフォーム | 状況 |
|---|---|
| Claude.ai（Web / アプリ） | ✅ 即日 |
| Claude API | ✅ 即日 |
| GitHub Copilot | ✅ 即日 |
| Amazon Bedrock / AWS | ✅ 即日 |
| Microsoft Azure AI Foundry | ✅ 即日 |

---

## 🗺️ どちらを使うべき？

```
難しい・長い・複雑 ────────────────▶ 🧚 Fable 5
                                      大規模コード生成
                                      多段階エージェント
                                      ゲーム・体験生成

簡単・短い・速さ優先 ───────────────▶ Opus 4.8
                                      日常チャット・QA
                                      ゼロ保持が必要な業務
                                      リアルタイム応答

コスト最優先・大量処理 ──────────────▶ Sonnet 4.6 / Haiku 4.5
```

> **実務推奨：** 8割の日常タスクは Opus 4.8、残り2割の重いジョブに Fable 5 をルーティングするのがコスパ最適。

---

## 🛡️ 安全機能

Fable 5 に組み込まれたクラシファイアが検知する領域：

- 🔐 **サイバーセキュリティ**（脆弱性悪用・攻撃ツール生成）
- 🧪 **生物・化学**（危険物質の合成）
- 📋 **蒸留**（モデル知識の不正複製）
- 🚨 **ジェイルブレイク検出**

検知時は Opus 4.8 が応答を代替するため、回答品質が著しく落ちることはありません。

---

## まとめ

Claude Fable 5 は Anthropic が長らく一般公開を控えてきた Mythos クラスの能力を、安全フィルター付きで初めて解放したモデルです。

長時間・複雑なタスクでは既存モデルを大きく上回り、特にコーディング・エージェント・クリエイティブ生成の分野で新しい可能性を広げます。

**6月22日まではサブスクユーザーが試せる絶好のタイミング。** クォータ消費が2倍になることを念頭に置きつつ、まずは重めのタスクで体験してみてください。

---

**参考リンク**
- [Claude Fable 5 and Claude Mythos 5 — Anthropic](https://www.anthropic.com/news/claude-fable-5-mythos-5)
- [Anthropic releases Claude Fable 5 — TechCrunch](https://techcrunch.com/2026/06/09/anthropic-released-claude-fable-5-its-most-powerful-model-publicly-days-after-warning-ai-is-getting-too-dangerous/)
- [Fable 5 can make video games with a click — TechCrunch](https://techcrunch.com/2026/06/09/anthropics-fable-5-can-make-weirdly-fun-video-games-with-the-click-of-a-button/)
- [Claude Fable 5 Costs Double Opus But Stays Free Until June 22 — Yellow.com](https://yellow.com/news/claude-fable-5-free-until-june-22)
- [Claude Fable 5 Pricing Explained — AY Automate](https://www.ayautomate.com/blog/claude-fable-5-pricing-explained)
- [Anthropic releases Mythos-like AI to the public — CNBC](https://www.cnbc.com/2026/06/09/anthropic-mythos-claude-fable-5.html)
- [Claude Fable 5 vs Opus 4.8 — Truefoundry](https://www.truefoundry.com/blog/claude-fable-5-vs-opus-4-8-benchmarks-pricing-when-to-use-each)
