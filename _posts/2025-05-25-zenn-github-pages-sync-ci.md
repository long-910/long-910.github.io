---
layout: post
title: "Zennの記事をGitHub Pagesに自動同期するCIの構築方法"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["github", "ci", "github-actions", "zenn", "github-pages"]
date: "2025-05-25 02:10"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2025-05-25-zenn-github-pages-sync-ci)でも公開しています。

# Zenn の記事を GitHub Pages に自動同期する CI の構築方法

## はじめに

Zenn で記事を書いていると、同じ内容を GitHub Pages のブログでも公開したい場合があります。この記事では、GitHub Actions を使用して、Zenn の記事を自動的に GitHub Pages のブログに同期する CI の構築方法について解説します。

## 前提条件

- Zenn の記事を管理する GitHub リポジトリ
- GitHub Pages のブログ用リポジトリ
- GitHub Personal Access Token (PAT)

## CI の構築手順

### 1. GitHub Personal Access Token の作成

1. GitHub の設定画面から「Developer settings」→「Personal access tokens」→「Tokens (classic)」を選択
2. 「Generate new token」をクリック
3. 以下の設定でトークンを作成：
   - Note: `Zenn to GitHub Pages Sync`
   - Expiration: 適切な有効期限を設定
   - Scopes: `repo`にチェック

### 2. リポジトリにシークレットを追加

1. Zenn の記事を管理するリポジトリの設定画面を開く
2. 「Secrets and variables」→「Actions」を選択
3. 「New repository secret」をクリック
4. 以下の設定でシークレットを追加：
   - Name: `GH_PAT`
   - Value: 先ほど作成した PAT を入力

### 3. GitHub Actions ワークフローの作成

`.github/workflows/sync-articles.yml`ファイルを作成し、以下の内容を記述します。
**以下の項目は必ず自分の環境に合わせて変更してください：**

1. **リポジトリ名の変更**:

```yaml
repository: long-910/long-910.github.io # 自分のGitHub Pagesリポジトリ名に変更
```

2. **Git 設定の変更**:

```yaml
git config --global user.name "long-910"  # 自分のGitHubユーザー名に変更
git config --global user.email "7323488+long-910@users.noreply.github.com"  # 自分のGitHub noreplyメールアドレスに変更
```

3. **Zenn のユーザー名の変更**:

```yaml
https://zenn.dev/long910/articles/ # 自分のZennユーザー名に変更
```

4. **画像パスの変更**:

```yaml
img_path: /assets/img/logos # 自分のブログの画像パスに変更
image:
  path: logo-only.svg # 自分のブログのデフォルト画像に変更
```

### 4. ワークフローの説明

このワークフローは以下のように動作します：

1. **トリガー**:

   - `main`ブランチへのプッシュ
   - `articles`ディレクトリ内のファイルが変更された場合のみ実行

2. **処理の流れ**:

   - ソースリポジトリ（Zenn 記事）をチェックアウト
   - ターゲットリポジトリ（GitHub Pages）をチェックアウト
   - Node.js 環境のセットアップ
   - zenn-cli のインストール
   - 記事ファイルの処理と同期
   - 変更のコミットとプッシュ

3. **記事の変換処理**:
   - ファイル名はそのまま保持
   - フロントマターの変換
   - 既存ファイルとの内容比較
   - 変更がある場合のみ更新

### 5. フロントマターの変換

Zenn のフロントマターを GitHub Pages の形式に変換します：

```yaml
# Zennのフロントマター
---
layout: post
title: "記事のタイトル"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["tag1", "tag2"]
date: "2024-03-21 12:00"
---
# 変換後のフロントマター
---
layout: post
title: "記事のタイトル"
---
```

変換の主なポイント：

1. **基本設定**:

   - `layout: post`を追加
   - タイトルはそのまま保持

2. **画像設定**:

   - Zenn のアイコンをデフォルト画像として設定
   - 画像パスとサイズを指定

3. **カテゴリーとタグ**:

   - `type`を`category`に変換
   - `topics`を`tags`に変換

4. **日付**:

   - `published_at`を`date`に変換

5. **Zenn へのリンク**:
   - GithubPagesの記事の先頭に Zenn の記事へのリンクを追加

### 6. ファイル処理の詳細

1. **新規ファイルの場合**:

   - AWKを使用してフロントマターを変換
   - そのままのファイル名でターゲットディレクトリにコピー

2. **既存ファイルの場合**:

   - 内容を比較して変更の有無を確認
   - 変更がある場合のみ更新
   - 変更がない場合はスキップ

3. **安全な処理**:
   - 一時ファイルを使用して変換処理
   - 処理後に一時ファイルを確実に削除
   - エラーハンドリングの実装

## 記事の作成ルール

この CI を正しく動作させるために、以下のルールに従って記事を作成する必要があります：

1. **ファイル名**:

   - 任意のファイル名を使用可能
   - 日本語ファイル名は避けることを推奨

2. **フロントマター**:
   - Zenn の形式のフロントマターを使用
   - 必要なメタデータは全て含める

## 動作確認方法

1. 新しい記事を作成してプッシュ
2. GitHub Actions の実行状況を確認
3. ターゲットリポジトリに記事が同期されていることを確認
4. 既存記事の更新をプッシュして、変更が正しく反映されることを確認

## 注意点

1. **セキュリティ**:

   - PAT は適切なスコープのみを付与
   - 定期的に PAT をローテーション

2. **ファイル名**:

   - 日本語ファイル名は使用しないことを推奨
   - スペースは使用しないことを推奨

3. **フロントマター**:
   - Zenn の形式を維持
   - 必要なメタデータは全て含める

## まとめ

この CI を導入することで、以下のメリットが得られます：

- 記事の一元管理が可能
- 手動での同期作業が不要
- 記事の公開プロセスが自動化
- ミスのリスクを低減
- 既存記事の更新を安全に処理
- 不要な更新を防止
- フロントマターの自動変換
- Zenn との相互リンク

## 参考リンク

- [GitHub Actions 公式ドキュメント](https://docs.github.com/ja/actions)
- [Zenn CLI 公式ドキュメント](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [GitHub Pages 公式ドキュメント](https://docs.github.com/ja/pages)
