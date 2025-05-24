---
layout: post
title: "ZennとGitHubの連携方法とzenn-cliの使い方"
img_path: /assets/img/screenshots
image:
  path: zenn.png
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: [ ["zenn", "github", "cli", "markdown"]]
date:  "2025-05-25 01:10"
---


---

この記事は[Zenn](https://zenn.dev/long_910/articles/)でも公開しています。

# Zenn と GitHub の連携方法と zenn-cli の使い方

## はじめに

Zenn は、エンジニア向けの技術記事プラットフォームです。GitHub との連携により、記事の管理や執筆がより効率的になります。この記事では、Zenn と GitHub の連携方法、および zenn-cli のインストールと使用方法について解説します。

## Zenn と GitHub の連携方法

### 1. GitHub リポジトリの準備

1. GitHub で新しいリポジトリを作成します
2. リポジトリの名前は任意ですが、例えば `zenn-articles` などが分かりやすいでしょう

### 2. Zenn のアカウント設定

1. [Zenn](https://zenn.dev)にログインします
2. 設定画面から「GitHub 連携」を選択します
3. 連携したい GitHub リポジトリを選択します

## zenn-cli のインストールと使用方法

### インストール方法

zenn-cli は、Node.js のパッケージマネージャーである npm を使用してインストールできます。

```bash
npm install -g zenn-cli
```

### 初期設定

1. リポジトリのルートディレクトリで以下のコマンドを実行します：

```bash
zenn init
```

これにより、以下のような構造が作成されます：

```
.
├── articles
│   └── .gitkeep
└── books
    └── .gitkeep
```

### 記事の作成

新しい記事を作成するには、以下のコマンドを使用します：

```bash
zenn new:article
```

このコマンドを実行すると、`articles`ディレクトリに新しい Markdown ファイルが作成されます。ファイル名は自動的に生成されますが、後で変更することも可能です。

### 記事のプレビュー

記事を執筆しながらプレビューを確認するには：

```bash
zenn preview
```

このコマンドを実行すると、ローカルサーバーが起動し、ブラウザで記事のプレビューを確認できます。

### 記事の構成

記事の冒頭には、以下のようなフロントマターを記述します：

```yaml
---
layout: post
title: "記事のタイトル"
img_path: /assets/img/screenshots
image:
  path: zenn.png
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: [ ["タグ1", "タグ2"]]
---
```

## 記事の公開

1. 記事を GitHub リポジトリにプッシュします
2. Zenn のダッシュボードで「GitHub から記事をインポート」を選択します
3. インポートしたい記事を選択して公開します

## まとめ

Zenn と GitHub の連携、および zenn-cli を使用することで、以下のようなメリットがあります：

- 記事のバージョン管理が容易になる
- ローカル環境で記事を執筆できる
- プレビュー機能で記事の確認が簡単
- 複数の記事を効率的に管理できる

## 参考リンク

- [Zenn 公式ドキュメント](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [GitHub 連携の設定方法](https://zenn.dev/zenn/articles/connect-to-github)
