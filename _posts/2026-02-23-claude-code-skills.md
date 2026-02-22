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
layout: post
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
layout: post
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
layout: post
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
layout: post
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
layout: post
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
layout: post
---
name: deploy
description: 本番環境へデプロイする
disable-model-invocation: true
---
layout: post
---
name: api-conventions
description: このプロジェクトのAPI設計規約。APIエンドポイントを書くときは必ず参照する。
user-invocable: false
---
layout: post
---

## 引数を使う

`$ARGUMENTS` でコマンドに渡した引数を受け取れます。

### 基本的な引数

```yaml
---
layout: post
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
layout: post
---

$0 コンポーネントを $1 から $2 へ移植してください。
既存の動作とテストをすべて保持してください。
```

```
/migrate SearchBar React Vue
```

→ `$0 = SearchBar`、`$1 = React`、`$2 = Vue` として展開されます。

---
layout: post
---
name: pr-review
description: 現在のPRをレビューする
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---
layout: post
---

## サブエージェントで独立実行する

`context: fork` を指定すると、Skill がメインの会話から切り離された独立したサブエージェントとして実行されます。

```yaml
---
layout: post
---

「$ARGUMENTS」について徹底的に調査してください。

1. 関連するファイルを Glob と Grep で検索
2. コードを読んで分析
3. 具体的なファイルパスと行番号を含めてまとめる
```

メインの会話のコンテキストを汚さずに、重い調査タスクを独立して走らせたいときに便利です。

---
layout: post
---
name: commit
description: 変更内容を分析してコミットメッセージを生成する
disable-model-invocation: true
---
layout: post
---
name: review
description: コードレビューを行う。セキュリティ・パフォーマンス・可読性を確認する。
---
layout: post
---
name: project-rules
description: このプロジェクトのコーディング規約と設計方針。コードを書くときは常に参照する。
user-invocable: false
---
layout: post
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
layout: post
---

## 参考リンク

- [Claude Code Skills 公式ドキュメント](https://code.claude.com/docs/en/slash-commands)
- [Agent Skills 標準仕様](https://agentskills.io)
