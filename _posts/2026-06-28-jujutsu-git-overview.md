---
layout: post
title: "Jujutsuとは？ Gitとの関係とGitHub連携をわかりやすく解説"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["jujutsu", "git", "versioncontrol", "github", "cli"]
date: "2026-06-28 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-28-jujutsu-git-overview)でも公開しています。


## Jujutsuとは

**Jujutsu**（読み方：じゅじゅつ、CLIコマンドは `jj`）は、Googleエンジニアによって開発されたオープンソースのバージョン管理システム（VCS）です。2022年ごろから公開され、2024年末には `jj-vcs` という独立したGitHub organizationに移管されました。

「GitのUXを根本的に改善する」というコンセプトを持ちつつ、**既存のGitリポジトリと完全互換**なのが最大の特徴です。

:::message
2026年現在、Jujutsuはまだ実験的なプロジェクトです。Git互換部分は安定して動作しており、多くの開発者が日常使いしていますが、一部機能は開発途中です。
:::

---

## GitとJujutsuの関係

Jujutsuは「Gitの代替」ではなく、**Gitの上に乗る新しいフロントエンド**と捉えるのが正確です。

```
┌─────────────────────────────┐
│       Jujutsu (jj)          │  ← 新しいUX・コマンド体系
├─────────────────────────────┤
│      Gitストレージ           │  ← データの実体はGitのまま
└─────────────────────────────┘
```

- `.git` ディレクトリはそのまま存在する
- 既存のGitリモート（GitHub/GitLab等）をそのまま使える
- `jj git fetch` / `jj git push` でGitリモートと同期できる
- チームの一部が `jj` を使い、残りが `git` を使うという混在運用も可能

---

## Gitとの主な概念の違い

### 1. ワーキングコピーが常にコミット

Gitでは「ワーキングツリー → ステージング → コミット」という3ステップがあります。

Jujutsuではこの概念が大きく変わります。

**ファイルを編集した瞬間、それは即座にコミットの一部になります。**

```
Git:    作業ディレクトリ → (git add) → ステージング → (git commit) → コミット
jj:     作業ディレクトリ ────────────────────────────────────────→ コミット（常時）
```

ステージング（`git add`）という概念がなく、`jj` は常にワーキングコピーを自動スナップショットしています。

### 2. ステージングエリアがない

`git add` は不要です。変更したファイルはすべて現在のコミット（`@` と表記）に自動的に含まれます。

```bash
# Git の場合
git add src/main.py
git commit -m "fix: バグ修正"

# jj の場合
jj commit -m "fix: バグ修正"   # add は不要
```

### 3. ブランチ → ブックマーク

Gitの「ブランチ」に相当するものをJujutsuでは**ブックマーク（bookmark）**と呼びます。

重要な違いは、Jujutsuでは**ブランチなしで作業するのが通常の状態**という点です。Gitの「detached HEAD」が通常状態に相当します。名前をつけるのはGitHub等にプッシュする直前でOKです。

```bash
# ブックマーク（ブランチ）を作成してプッシュ
jj bookmark create feature-login -r @-
jj git push --bookmark feature-login --allow-new
```

### 4. Change IDとCommit ID

Jujutsuのコミットには2種類のIDがあります。

| ID | 説明 |
|---|---|
| **Change ID** | そのコミットを「書き換えても」変わらない識別子（例: `qmvvknzz`） |
| **Commit ID** | Gitのcommit hashと同じ。書き換えると変わる |

これにより、`git commit --amend` で書き換えたコミットを後から参照するときに混乱が起きません。

### 5. 匿名ブランチ（Anonymous Branches）

Jujutsuは**匿名ブランチを正式にサポート**しています。名前をつけないまま複数のコミットを積み重ねられます。GitがMercurial的な設計思想も取り込んだ形です。

### 6. ファーストクラスのコンフリクト処理

Gitではコンフリクトが発生すると作業がブロックされます。

Jujutsuでは**コンフリクト状態のままコミットを持ち続けられます**。

```bash
jj rebase -d main   # コンフリクトが発生してもそのまま続行できる
jj status           # コンフリクトがある旨が表示されるが、作業は続けられる
jj resolve          # 準備ができたときに解決する
```

さらに、コンフリクトを解決すると**その解決がすべての子孫コミットに自動で伝播**されます。

### 7. オペレーションログ（Operation Log）

リポジトリに対するすべての操作が記録されます。

```bash
jj op log     # 過去のすべての操作を表示
jj undo       # 直前の操作を取り消す
jj op restore abc123   # 特定の状態に戻す
```

`git reflog` より強力で、`git fetch` や `git push` まで含めた全操作が対象です。

---

## GItとのコマンド対応表

| 目的 | Git | Jujutsu |
|---|---|---|
| リポジトリ作成 | `git init` | `jj git init` |
| クローン | `git clone <url>` | `jj git clone <url>` |
| 状態確認 | `git status` | `jj status` |
| ログ確認 | `git log` | `jj log` |
| コミット | `git add . && git commit -m "msg"` | `jj commit -m "msg"` |
| コミットメッセージ編集 | `git commit --amend` | `jj describe` |
| 新しい作業開始 | `git checkout -b <branch>` | `jj new` |
| 別コミットに移動 | `git checkout <hash>` | `jj edit <id>` |
| コミットをまとめる | `git rebase -i` | `jj squash` |
| コミットを分割 | `git rebase -i` | `jj split` |
| リベース | `git rebase <branch>` | `jj rebase -d <branch>` |
| フェッチ | `git fetch` | `jj git fetch` |
| プッシュ | `git push` | `jj git push` |
| 取り消し | `git reflog` + `git reset` | `jj undo` |
| ブランチ作成 | `git checkout -b <name>` | `jj bookmark create <name>` |

---

## GitHubとの連携

### リポジトリのクローン

```bash
jj git clone https://github.com/user/repo.git
cd repo
```

### 日常の開発フロー

```bash
# 1. mainから新しい変更を始める
jj new main

# 2. ファイルを編集（自動的に現在のコミットに反映される）
vim src/feature.py

# 3. コミットメッセージを付けて次のコミットへ進む
jj commit -m "feat: 新機能を追加"

# 4. さらに変更を続ける...
```

### GitHubへのプッシュとPull Request

```bash
# ブックマーク（ブランチ）を作成
jj bookmark create feature-my-function -r @-

# GitHubへプッシュ（初回は --allow-new が必要）
jj git push --bookmark feature-my-function --allow-new

# あとは GitHub上でPull Requestを作成（通常通り）
```

または、ブックマーク名を自動生成させることもできます：

```bash
# Change IDから自動でブックマーク名を生成してプッシュ
jj git push --change @-
```

### リモートの変更を取り込む

```bash
# フェッチ（git pull に相当する直接コマンドはない）
jj git fetch

# mainに追従してリベース
jj rebase -d main
```

### レビューコメントへの対応

```bash
# 方法1: 新しいコミットを追加する（コミット履歴に残る方式）
jj new feature-my-function
# 修正を加える
jj commit -m "fix: レビュー指摘を修正"
jj bookmark set feature-my-function   # ブックマークを移動
jj git push

# 方法2: 既存コミットを書き換える（履歴をクリーンに保つ方式）
jj edit <change-id>   # 対象コミットに移動
# 修正を加える（子孫は自動でリベースされる）
jj git push --force-with-lease
```

### スタック型PRの作成

複数の依存するPRを同時に作る場合：

```bash
jj new main
# PR1の変更...
jj commit -m "feat: 基盤実装"

jj new
# PR2の変更（PR1に依存）...
jj commit -m "feat: 追加機能"

# それぞれブックマークを作成してプッシュ
jj bookmark create part1 -r @--
jj bookmark create part2 -r @-
jj git push --bookmark part1 --allow-new
jj git push --bookmark part2 --allow-new
```

---

## Jujutsuを使い始める

### インストール

```bash
# macOS (Homebrew)
brew install jj

# Cargo (Rust)
cargo install --locked jj-bin

# Linux (各ディストリビューションのパッケージマネージャーも利用可)
```

### 既存のGitリポジトリで試す

```bash
cd existing-git-repo
jj git init --git-repo .   # Gitリポジトリとして初期化
jj log                      # ログを確認
```

既存の `.git` ディレクトリをそのまま使うため、`git` コマンドとの混在運用も問題なく動作します。

---

## Revsets：強力なコミット選択言語

Jujutsuには**Revsets**という関数型のクエリ言語があります。

```bash
# mainより新しい自分のコミットをすべて表示
jj log -r "ancestors(@) & descendants(main) & mine()"

# コンフリクトのあるコミットを表示
jj log -r "conflict()"

# 直近5件のコミット
jj log -r "heads(ancestors(@, 5))"
```

`git log` のオプションより表現力が高く、複雑な条件のコミット群を簡単に選択できます。

---

## まとめ：Jujutsuを使うべきか？

### こんな人におすすめ

- `git rebase -i` を頻繁に使う人
- コミット履歴をきれいに保ちたい人
- `git stash` / `git worktree` の代替を探している人
- スタック型PRを多用する人

### 注意点

- まだ実験的なプロジェクト（ただし実用レベルで安定）
- エディタやIDEとの統合はGitより少ない
- チームメンバー全員が移行する必要はない（混在可能）
- `git pull` 相当の単一コマンドがない（`jj git fetch` + `jj rebase` の2ステップ）

Jujutsuは「Gitを捨てる」ツールではなく、**Gitの資産を活かしながらUXを改善する**ツールです。個人の開発環境から気軽に試せるので、まず `jj git clone` で既存のリポジトリをクローンして触ってみるのがおすすめです。

---

## 参考リンク

- [Jujutsu 公式リポジトリ (jj-vcs/jj)](https://github.com/jj-vcs/jj)
- [Jujutsu公式ドキュメント](https://docs.jj-vcs.dev/)
- [GitHubとの連携ガイド（公式）](https://docs.jj-vcs.dev/latest/github/)
- [GitとJujutsuの比較（公式）](https://docs.jj-vcs.dev/latest/git-comparison/)
