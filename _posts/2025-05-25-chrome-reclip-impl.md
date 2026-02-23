---
layout: post
title: "Chrome拡張機能「ReClip」の開発記録"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["chrome-extension", "typescript", "vite", "i18n", "cursor"]
date: "2025-05-25 20:42"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2025-05-25-chrome-reclip-impl)でも公開しています。


# Chrome 拡張機能「ReClip」の開発記録

## はじめに

YouTube の動画リンクを簡単に保存して後で見返せる Chrome 拡張機能「ReClip」を開発しました。このプロジェクトは、AI を活用したコードエディタ「Cursor」を使用して開発を行いました。この記事では、開発の背景、機能、開発過程、そして得られた成果について詳しく解説します。

## 開発背景

YouTube で面白い動画を見つけた時、後で見返したいものをログインなしでシンプルに保存したい。

そのため、以下の要件を満たす Chrome 拡張機能の開発を決意しました：

- YouTube の動画リンクを右クリックで簡単に保存
- 保存した動画を一覧で確認可能
- セキュリティを考慮したパスワード保護機能
- 多言語対応によるグローバルな利用

## 主な機能

### 1. 動画の保存と管理

- 右クリックメニューから「Save to ReClip」で簡単に保存
- ポップアップで保存した動画の一覧表示
- 個別または一括での動画削除機能

### 2. セキュリティ機能

- パスワード保護による安全なアクセス制御
- セッション管理による利便性の向上
- パスワード変更機能
- セッション時間のカスタマイズ（1 分〜24 時間）

### 3. 多言語対応

- 英語、日本語、中国語の 3 言語をサポート
- 言語設定の切り替え機能

### 4. その他の機能

- コンソールログ出力の切り替え
- シンプルで使いやすい UI

## 技術スタック

- **言語**: TypeScript
- **ビルドツール**: Vite
- **パッケージ管理**: npm
- **その他**: Chrome Extension Manifest V3

## 開発過程

### 1. Cursor を活用した開発環境の構築

Cursor は、AI を活用したコードエディタとして、以下のような利点を提供してくれました：

- AI によるコード補完と提案機能
- 自然言語でのコード生成
- リアルタイムのエラー検出と修正提案
- コードの最適化提案

開発環境の構築は以下の手順で行いました：

1. Cursor のインストールと初期設定
2. TypeScript プロジェクトの作成
3. 必要なパッケージのインストール
4. Vite の設定
5. Chrome 拡張機能の基本構造の構築

### 2. Cursor を活用した開発フロー

#### コード生成と補完

- AI によるコード生成機能を活用し、基本的なコンポーネントの実装を効率化
  - コンポーネントの基本構造の自動生成
  - TypeScript の型定義の自動生成
  - インターフェースの設計支援
  - テストコードの自動生成
- 自然言語での要件定義からコードへの変換
  - 「YouTube の動画情報を取得する関数を作成して」などの自然言語での指示
  - コンテキストを考慮した適切なコード生成
  - 既存コードとの整合性を考慮した実装
- コードレビューと改善提案の自動化
  - ベストプラクティスに基づいた改善提案
  - セキュリティ考慮事項の自動チェック
  - パフォーマンス最適化の提案

#### デバッグと最適化

- リアルタイムのエラー検出
  - TypeScript の型エラーの即時検出
  - ランタイムエラーの予測と防止
  - 非同期処理のエラーハンドリングの提案
- パフォーマンス最適化の提案
  - 不要なレンダリングの検出
  - メモリリークの可能性の指摘
  - 非同期処理の最適化提案
- コードの品質向上のための提案
  - コードの可読性向上の提案
  - 命名規則の一貫性チェック
  - コメントの適切性チェック

#### リファクタリング

- AI によるコードの改善提案
  - 重複コードの検出と統合
  - デザインパターンの適用提案
  - アーキテクチャの最適化
- コードの整理と構造化
  - 関連する機能のグループ化
  - モジュール分割の提案
  - 依存関係の整理
- テストカバレッジの向上
  - テストケースの自動生成
  - エッジケースの提案
  - テストの品質向上

### 3. アーキテクチャ設計

プロジェクトは以下のような構造で設計しました：

```
src/
├── background/    # バックグラウンドスクリプト
├── content/       # コンテンツスクリプト
├── popup/         # ポップアップUI
├── options/       # 設定画面
├── i18n/          # 多言語対応
└── utils/         # ユーティリティ関数
```

### 4. 実装のポイント

#### Cursor を活用した効率的な開発

- AI によるコード生成を活用した迅速なプロトタイピング
- 自然言語での要件定義からコードへの変換
- コードレビューと改善提案の自動化

#### パスワード保護機能

- セキュアなパスワードハッシュ化
- セッション管理による再認証の最小化
- 設定可能なセッション時間

#### 多言語対応

- i18n システムの実装
- 言語ファイルの管理
- 動的な言語切り替え

#### UI/UX

- シンプルで直感的なインターフェース
- レスポンシブなデザイン
- アクセシビリティへの配慮

## 成果と学び

### 技術的な成果

- Chrome 拡張機能の開発フローを確立
- TypeScript による型安全な開発
- 効率的なビルドシステムの構築
- Cursor を活用した効率的な開発手法の確立

### Cursor を活用した開発の利点

- 開発速度の向上
- コード品質の改善
- 学習コストの削減
- 効率的なデバッグと最適化

### ユーザー体験の向上

- 直感的な操作性
- セキュリティと利便性の両立
- グローバルなユーザーサポート

### 今後の展望

- より多くの言語への対応
- 動画のメモ機能の追加
- カテゴリ分け機能の実装
- クラウド同期機能の検討
- Cursor の新機能を活用した開発効率のさらなる向上

## まとめ

ReClip の開発を通じて、Chrome 拡張機能の開発手法、セキュリティ考慮事項、多言語対応の実装方法など、多くの技術的な知見を得ることができました。また、Cursor を活用した開発により、効率的なコーディングと品質の高いコードの実現が可能になりました。

特に、AI を活用した開発環境の利点を最大限に活かすことで、開発効率とコード品質の両方を向上させることができました。今後もユーザーフィードバックを基に、機能の改善と拡張を続けていきたいと考えています。

## 参考リンク

- [GitHub リポジトリ](https://github.com/long-910/chrome-reclip)
- [Chrome Web Store](https://chromewebstore.google.com/detail/reclip/pgkejilkahilhncnighindfakoecmcii?authuser=0&hl=ja)
- [Cursor - AI を活用したコードエディタ](https://cursor.sh)

### 具体的な開発事例

#### パスワード保護機能の実装

```typescript
// Cursorによる提案を受けて実装したパスワードハッシュ化関数
async function hashPassword(password: string): Promise<string> {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hash = await crypto.subtle.digest("SHA-256", data);
  return Array.from(new Uint8Array(hash))
    .map((b) => b.toString(16).padStart(2, "0"))
    .join("");
}
```

#### 多言語対応の実装

```typescript
// Cursorの提案により実装したi18nシステム
interface I18nConfig {
  locale: string;
  messages: Record<string, string>;
}

class I18nManager {
  private static instance: I18nManager;
  private config: I18nConfig;

  private constructor() {
    this.config = {
      locale: "en",
      messages: {},
    };
  }

  static getInstance(): I18nManager {
    if (!I18nManager.instance) {
      I18nManager.instance = new I18nManager();
    }
    return I18nManager.instance;
  }

  setLocale(locale: string): void {
    this.config.locale = locale;
  }

  translate(key: string): string {
    return this.config.messages[key] || key;
  }
}
```

#### UI コンポーネントの実装

```typescript
// Cursorの提案により実装した動画リストコンポーネント
interface VideoItem {
  id: string;
  title: string;
  thumbnail: string;
  url: string;
}

class VideoList extends HTMLElement {
  private videos: VideoItem[] = [];

  constructor() {
    super();
    this.attachShadow({ mode: "open" });
  }

  connectedCallback() {
    this.render();
  }

  private render() {
    if (!this.shadowRoot) return;

    this.shadowRoot.innerHTML = `
      <style>
        .video-list {
          display: grid;
          grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
          gap: 1rem;
          padding: 1rem;
        }
        .video-item {
          border: 1px solid #ddd;
          border-radius: 4px;
          padding: 0.5rem;
        }
      </style>
      <div class="video-list">
        ${this.videos
          .map(
            (video) => `
          <div class="video-item">
            <img src="${video.thumbnail}" alt="${video.title}">
            <h3>${video.title}</h3>
          </div>
        `
          )
          .join("")}
      </div>
    `;
  }
}

customElements.define("video-list", VideoList);
```

### 開発効率の向上

#### コード生成の効率化

- 自然言語での要件定義からコードへの変換
  - コンポーネントの基本構造の自動生成
  - 型定義の自動生成
  - テストコードの自動生成
- コードの再利用性の向上
  - 共通コンポーネントの自動生成
  - ユーティリティ関数の提案
  - デザインパターンの適用

#### デバッグの効率化

- リアルタイムのエラー検出
  - 型エラーの即時検出
  - ランタイムエラーの予測
  - 非同期処理のエラーハンドリング
- パフォーマンス最適化
  - ボトルネックの検出
  - 最適化提案の自動生成
  - メモリ使用量の最適化

#### ドキュメント生成

- コードの自動ドキュメント化
  - JSDoc コメントの自動生成
  - API ドキュメントの生成
  - 使用例の自動生成
- 変更履歴の管理
  - コミットメッセージの提案
  - 変更内容の要約
  - レビューコメントの生成
