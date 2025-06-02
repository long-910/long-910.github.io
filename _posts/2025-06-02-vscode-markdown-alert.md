---
layout: post
title: "VSCodeでGitHubスタイルのアラートを実現する「Markdown Alert」拡張機能の使い方"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["vscode", "markdown", "github", "拡張機能"]
date: "2025-06-02 20:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2025-06-02-vscode-markdown-alert)でも公開しています。

# VSCode で GitHub スタイルのアラートを実現する「Markdown Alert」拡張機能の使い方

> [!NOTE]
> この記事は 2025 年 6 月 2 日に公開されました。

## はじめに

GitHub の Markdown で使用できるアラート構文を VSCode でプレビューできる「Markdown Alert」拡張機能について紹介します。この拡張機能を使用することで、GitHub のドキュメントでよく見かける警告や注意書きなどのアラートを、VSCode 上で簡単に作成・プレビューすることができます。

![拡張機能のプレビュー例](https://github.com/ByPikod/vscode-markdown-alert/raw/HEAD/promotions/preview.png)

## なぜこの拡張機能が必要なのか？

GitHub のドキュメントでは、重要な情報を目立たせるためにアラート構文が使用されています。しかし、VSCode の標準の Markdown プレビューでは、これらのアラートが正しく表示されません。この拡張機能を使用することで、以下のメリットがあります：

- ドキュメント作成時にリアルタイムでアラートの見た目を確認できる
- GitHub のドキュメントと同じ見た目でプレビューできる
- チーム内でのドキュメント共有時に一貫性を保てる

## インストール方法

### VSCode 拡張機能タブからインストール

1. VSCode を開く
2. 拡張機能タブ（Ctrl+Shift+X）を開く
3. 検索バーに「Markdown Preview for Github Alerts」と入力
4. インストールボタンをクリック

### Marketplace から直接インストール

1. [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=yahyabatulu.vscode-markdown-alert)にアクセス
2. 「Install」ボタンをクリック
3. VSCode が自動的に起動し、インストールの確認ダイアログが表示される
4. 「Install」をクリックしてインストールを完了

### コマンドラインからインストール

```bash
code --install-extension yahyabatulu.vscode-markdown-alert
```

> [!TIP]
> インストール後、VSCode の再起動は必要ありません。すぐに使用を開始できます。

表示例：

![](https://storage.googleapis.com/zenn-user-upload/5e80168ca2c5-20250602.png)

## 基本的な使い方

以下のような構文でアラートを作成できます：

```markdown
> [!NOTE]
> これは注意書きです。一般的な情報や補足説明に使用します。

> [!TIP]
> これはヒントです。便利な使い方や推奨事項を示す際に使用します。

> [!IMPORTANT]
> これは重要な情報です。必ず読者に伝えたい重要なポイントに使用します。

> [!WARNING]
> これは警告です。潜在的な問題や注意が必要な点を示す際に使用します。

> [!CAUTION]
> これは注意喚起です。特に慎重な対応が必要な場合に使用します。
```

表示例：

![](https://storage.googleapis.com/zenn-user-upload/ad0fbcaa9d1d-20250602.png)

## アラートのカスタマイズ

### スタイルのカスタマイズ

VSCode の設定で、アラートのスタイルをカスタマイズすることができます：

```json
{
  "markdown-alert.styles": {
    "note": {
      "icon": "📝",
      "color": "#0366d6",
      "backgroundColor": "#f1f8ff"
    },
    "warning": {
      "icon": "⚠️",
      "color": "#d73a49",
      "backgroundColor": "#ffeef0"
    }
  }
}
```

### ショートカットの活用

VSCode のスニペット機能を使用して、アラートを素早く挿入することができます：

```json
{
  "Markdown Alert Note": {
    "prefix": "alert-note",
    "body": ["> [!NOTE]", "> $1"],
    "description": "Insert a note alert"
  }
}
```

## 実践的な使用例

### プロジェクトドキュメントでの使用例

```markdown
# プロジェクトのセットアップ

> [!IMPORTANT]
> このプロジェクトを実行するには、Node.js v18 以上が必要です。

> [!TIP]
> 開発環境のセットアップは `npm install` で完了します。

> [!WARNING]
> 本番環境では必ず環境変数を設定してください。
```

表示例：

![](https://storage.googleapis.com/zenn-user-upload/03cef5f20240-20250602.png)

### API ドキュメントでの使用例

```markdown
# API リファレンス

> [!NOTE]
> この API は現在ベータ版です。本番環境での使用は推奨されません。

> [!CAUTION]
> レート制限: 1 分間に 100 リクエストまで

> [!TIP]
> エラーハンドリングの実装例は `examples/` ディレクトリを参照してください。
```

表示例：

![](https://storage.googleapis.com/zenn-user-upload/15381ea8807d-20250602.png)

## トラブルシューティング

### よくある問題と解決方法

1. アラートが表示されない場合

   - VSCode を再起動する
   - 拡張機能が正しくインストールされているか確認する
   - Markdown ファイルの拡張子が `.md` であることを確認する

2. スタイルが適用されない場合
   - 設定ファイルの構文が正しいか確認する
   - カスタムスタイルの設定をリセットする

## まとめ

Markdown Alert 拡張機能を使用することで、GitHub スタイルのアラートを簡単に作成できます。ドキュメントの可読性を向上させ、重要な情報を効果的に伝えることができます。

> [!NOTE]
> この拡張機能は[Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=yahyabatulu.vscode-markdown-alert)から入手できます。

## 参考リンク

- [GitHub Docs - Alerts](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#alerts)
- [VSCode Markdown Preview](https://code.visualstudio.com/docs/languages/markdown)
- [拡張機能の GitHub リポジトリ](https://github.com/ByPikod/vscode-markdown-alert)
