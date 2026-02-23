---
layout: post
title: "Claude Code のSkillで自作スラッシュコマンドを作ろう"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["claudecode", "claude", "ai", "cli", "productivity"]
date: "2026-02-23 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-02-23-claude-code-skills)でも公開しています。


Claude Code を使っていると、同じような指示を何度も打ち込む場面があります。「このプロジェクトのコーディング規約に沿ってコードレビューして」「コミットメッセージを日本語で書いて」──こういった定型作業を `/review` や `/commit` の一言で呼び出せたら便利ですよね。

それを実現するのが **Skill（スキル）** です。

自分も最近使い始めたところですが、思ったより簡単に作れて、繰り返し作業の省力化にかなり効果がありそうなので記事にまとめました。

---

## Skill とは何か

Skill は、Claude Code に登録できる**カスタムスラッシュコマンド**です。

マークダウンファイル（`SKILL.md`）に「このコマンドを呼んだら何をするか」を書いておくだけで、`/コマンド名` で呼び出せるようになります。

### スラッシュコマンドとの関係

以前は「Custom Slash Commands」として `.claude/commands/review.md` のような形式もありましたが、現在は Skill システムに統合されています。どちらの形式も動作しますが、今は `.claude/skills/review/SKILL.md` の形式が推奨です。

### Skill でできること

- **定型作業を一言で呼び出す**（例：`/commit` でコミットメッセージを生成）
- **プロジェクト固有のルールを組み込む**（例：チームのコーディング規約を毎回読ませる）
- **動的な値を渡す**（例：`/fix-issue 123` でIssue番号を渡す）
- **サブエージェントで独立実行**（例：重い調査タスクをバックグラウンドで走らせる）

---

## 前提・環境

| 項目 | 内容 |
|---|---|
| 必要なもの | Claude Code がインストールされていること |
| 知識 | マークダウンが書ければOK |
| 難易度 | 初心者でも作れる |

Claude Code のインストールがまだの方は、以下を参考にしてください。

```bash
npm install -g @anthropic-ai/claude-code
```

---

## Skill の保存場所と適用範囲

Skill には3つの保存場所があり、**どこに置くかで適用範囲が変わります**。

| 場所 | パス | 範囲 |
|---|---|---|
| 個人（グローバル） | `~/.claude/skills/<スキル名>/SKILL.md` | 自分のすべてのプロジェクト |
| プロジェクト | `.claude/skills/<スキル名>/SKILL.md` | そのプロジェクトのみ |
| エンタープライズ | 管理者が一括配布 | 組織全体のユーザー |

**優先順位**：エンタープライズ ＞ 個人 ＞ プロジェクト（同名の場合）

まずは個人の `~/.claude/skills/` に作るのが手軽です。チームで共有したい場合はプロジェクトの `.claude/skills/` に置いてgitで管理します。

---

## はじめての Skill を作る

### 基本構造

Skill は1ディレクトリ1コマンドの構造になっています。

```
~/.claude/skills/
└── explain/        ← コマンド名（/explain で呼び出せる）
    └── SKILL.md    ← 必須ファイル
```

`SKILL.md` は2つのパートで構成されます。

```markdown
---
name: explain
description: コードを初心者にもわかりやすく説明する
---

コードを説明するときは以下の手順で行ってください：

1. **まず比喩で説明する**: 日常生活のものに例える
2. **図を描く**: ASCII アートでフローや構造を示す
3. **ステップごとに解説**: 何が起きているか順番に説明
4. **よくある誤解を指摘**: つまずきやすいポイントを挙げる
```

- `---` で囲まれた部分が **フロントマター**（設定）
- その下が Claude への **指示内容**

### 実際に動かしてみる

```bash
# ディレクトリを作成
mkdir -p ~/.claude/skills/explain

# SKILL.md を作成
cat > ~/.claude/skills/explain/SKILL.md << 'EOF'
---
name: explain
description: コードを初心者にもわかりやすく説明する
---

コードを説明するときは以下の手順で行ってください：

1. **まず比喩で説明する**: 日常生活のものに例える
2. **図を描く**: ASCII アートでフローや構造を示す
3. **ステップごとに解説**: 何が起きているか順番に説明
4. **よくある誤解を指摘**: つまずきやすいポイントを挙げる

説明は会話的に、専門用語はできるだけ避けてください。
EOF
```

Claude Code を再起動すると `/explain` が使えるようになります。

```
/explain この関数の処理を教えて
```

---

## フロントマターの設定項目

`SKILL.md` の冒頭に書く YAML で動作を細かく制御できます。

```yaml
---
name: review          # コマンド名（省略時はディレクトリ名）
description: |        # どんな時に使うか（Claude が自動判断にも使う）
  コードレビューを行う。
  コーディング規約・セキュリティ・パフォーマンスを確認する。
argument-hint: "[対象ファイル]"   # 引数のヒント（オートコンプリートに表示）
allowed-tools: Read, Grep, Glob  # このSkillで使えるツールを制限
context: fork                    # サブエージェントで独立実行
agent: Explore                   # context: fork のときに使うエージェント種別
disable-model-invocation: true   # Claude が自動で使わないようにする
user-invocable: false            # ユーザーが / メニューから呼べないようにする
---
```

### よく使う設定の組み合わせ

**ユーザーが手動で呼ぶ用（自動実行させたくない）:**

```yaml
---
name: deploy
description: 本番環境へデプロイする
disable-model-invocation: true
---
```

`disable-model-invocation: true` を付けると、「デプロイして」とチャットで頼んでも Claude が勝手に `/deploy` を呼ばなくなります。副作用のある操作に使いましょう。

**Claude が文脈を見て自動で使う用:**

```yaml
---
name: api-conventions
description: このプロジェクトのAPI設計規約。APIエンドポイントを書くときは必ず参照する。
user-invocable: false
---
```

`user-invocable: false` にすると `/` メニューから消えますが、Claude が必要と判断したときに自動で読み込みます。プロジェクトのルール集などに使うと便利です。

---

## 引数を使う

`$ARGUMENTS` でコマンドに渡した引数を受け取れます。

### 基本的な引数

```yaml
---
name: fix-issue
description: GitHubのIssueを修正する
disable-model-invocation: true
---

GitHub Issue #$ARGUMENTS を修正してください。

1. Issue の内容を確認する
2. 修正箇所を特定する
3. 実装する
4. テストを書く
5. コミットを作成する（コミットメッセージは日本語で）
```

```
/fix-issue 123
```

→ Claude は「GitHub Issue #123 を修正してください。...」というプロンプトを受け取ります。

### 複数の引数（位置引数）

```yaml
---
name: migrate
description: コンポーネントをフレームワーク間で移植する
---

$0 コンポーネントを $1 から $2 へ移植してください。
既存の動作とテストをすべて保持してください。
```

```
/migrate SearchBar React Vue
```

→ `$0 = SearchBar`、`$1 = React`、`$2 = Vue` として展開されます。

---

## シェルコマンドで動的な情報を注入する

`` !`コマンド` `` 構文を使うと、Skill が呼ばれた瞬間にシェルコマンドを実行してその結果を埋め込めます。

```yaml
---
name: pr-review
description: 現在のPRをレビューする
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## PRの情報

- 差分: !`gh pr diff`
- コメント: !`gh pr view --comments`
- 変更ファイル: !`gh pr diff --name-only`

上記の情報を元に、以下の観点でレビューしてください：
- バグの可能性
- セキュリティ上の問題
- パフォーマンスへの影響
- テストの網羅性
```

`/pr-review` を呼ぶだけで、最新のPR情報が自動的に取得されてレビューが始まります。

---

## サブエージェントで独立実行する

`context: fork` を指定すると、Skill がメインの会話から切り離された独立したサブエージェントとして実行されます。

```yaml
---
name: deep-research
description: テーマを徹底的に調査する
context: fork
agent: Explore
---

「$ARGUMENTS」について徹底的に調査してください。

1. 関連するファイルを Glob と Grep で検索
2. コードを読んで分析
3. 具体的なファイルパスと行番号を含めてまとめる
```

メインの会話のコンテキストを汚さずに、重い調査タスクを独立して走らせたいときに便利です。

---

## 実用的なSkillの例

### コミットメッセージを生成する

```markdown
---
name: commit
description: 変更内容を分析してコミットメッセージを生成する
disable-model-invocation: true
---

ステージングされた変更を確認し、コミットメッセージを生成してください。

## ルール
- プレフィックス: feat / fix / docs / refactor / test / chore
- 件名は日本語で50文字以内
- 本文は変更の「なぜ」を説明する
- フォーマット:
  ```
  <prefix>: <件名>

  <本文（任意）>
  ```

コミットする前に必ずユーザーに確認してください。
```

### コードレビュー

```markdown
---
name: review
description: コードレビューを行う。セキュリティ・パフォーマンス・可読性を確認する。
---

以下の観点でコードレビューを行ってください：

## チェック項目
1. **バグ**: ロジックの誤り、エッジケースの漏れ
2. **セキュリティ**: SQLインジェクション、XSS、認証の抜け
3. **パフォーマンス**: 不要なループ、N+1問題、メモリリーク
4. **可読性**: 命名、関数の長さ、コメントの有無
5. **テスト**: 網羅性、エッジケースのテスト

## 出力形式
各問題について：
- 🔴 重大 / 🟡 中程度 / 🟢 軽微 で重要度を示す
- 問題の場所（ファイル名:行番号）
- 問題の説明
- 改善案

$ARGUMENTS
```

### このプロジェクトのルールを常に適用する

```markdown
---
name: project-rules
description: このプロジェクトのコーディング規約と設計方針。コードを書くときは常に参照する。
user-invocable: false
---

## コーディング規約
- TypeScript を使用、型の `any` は禁止
- 関数名はキャメルケース、定数はアッパースネークケース
- 1関数は50行以内を目安に

## コミット規約
- Conventional Commits に従う
- コミットメッセージは日本語

## ディレクトリ構造
- `src/components/` — UIコンポーネント
- `src/hooks/` — カスタムフック
- `src/utils/` — ユーティリティ関数
```

`.claude/skills/project-rules/SKILL.md` に置いてgitで管理すると、チームメンバー全員で共有できます。

---

## スキルの確認方法

### 登録済みのスキルを見る

Claude Code のセッション内で `/` を入力すると、使えるスキルの一覧が補完として表示されます。

### デバッグ

スキルが期待通りに動かない場合：

1. `SKILL.md` のYAML構文エラーがないか確認（インデントに注意）
2. ファイルのパスが正しいか確認（`SKILL.md` は大文字）
3. Claude Code を再起動してみる

---

## まとめ

| やりたいこと | 設定 |
|---|---|
| ユーザーが手動で呼ぶコマンドを作る | `disable-model-invocation: true` |
| Claudeが自動で使うルール集を作る | `user-invocable: false` |
| 引数を受け取る | `$ARGUMENTS` または `$0`, `$1`... |
| シェルコマンドの結果を埋め込む | `` !`command` `` |
| サブエージェントで独立実行 | `context: fork` |
| チームで共有する | `.claude/skills/` をgit管理 |

まずは `/commit` や `/review` のような、自分がよく使う操作から作り始めるのがおすすめです。一度作ると毎回の指示入力が不要になるので、地味にストレスが減ります。

---

## 参考リンク

- [Claude Code Skills 公式ドキュメント](https://code.claude.com/docs/en/slash-commands)
- [Agent Skills 標準仕様](https://agentskills.io)
