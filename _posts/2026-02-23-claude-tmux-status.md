---
layout: post
title: "Claude Code の使用量をtmuxのステータスバーでリアルタイム監視する「claude-tmux-status」を作った"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["claudecode", "claude", "tmux", "python", "個人開発"]
date: "2026-02-23 13:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-02-23-claude-tmux-status)でも公開しています。

Claude Code を使っていると「気づいたら使用量が上限に達していた」という経験、ありませんか？

作業に集中しているとつい忘れてしまう使用量の確認。かといって別のウィンドウを開いて確認するのも手間です。

そこで、**tmuxのステータスバーに常時表示**する「claude-tmux-status」を作りました。

https://github.com/long-910/claude-tmux-status

---
layout: post
---

## なぜ作ったか

Claude Code の使用量制限は「5時間のローリングウィンドウ」と「週間制限」の2つで管理されています（詳細は[こちらの記事](https://zenn.dev/long910/articles/2026-02-21-claude-code-usage-limit)を参照）。

問題は、この使用量が**どこにも常時表示されない**ことです。

- 上限に達してから初めて「あ、使いすぎた」と気づく
- 別のブラウザタブやコマンドを都度開いて確認するのが面倒
- 作業フローが途切れる

tmuxユーザーであれば、ステータスバーには常にCPU使用率・時刻・バッテリー残量などを表示しているはずです。**Claude Code の使用量もそこに並べてしまおう**というのが発想の出発点です。

---
layout: post
---

## 必要環境

- Python 3.10 以上
- tmux 3.0 以上
- Claude Code インストール済み（`~/.claude/.credentials.json` が存在すること）

---
layout: post
---

## 使い方

### コマンド一覧

| コマンド              | 説明                                    |
| --------------------- | --------------------------------------- |
| `claude-usage`        | tmux ステータスバー用のコンパクト表示   |
| `claude-usage toggle` | パーセントモード↔コストモードを切り替え |
| `claude-usage long`   | プログレスバー付きの詳細表示            |
| `claude-usage json`   | JSON形式で全データ出力                  |

### キーバインド

tmux内で `Prefix + U` を押すと、パーセントモードとコストモードがトグルで切り替わります。

### 表示モードの永続化

選択した表示モードは `~/.claude/tmux-display-mode` に保存されるため、tmuxセッションをまたいでも設定が引き継がれます。

---
layout: post
---

## ccusage との違い

Claude Code の使用量を可視化するツールとして [ccusage](https://github.com/ryoppippi/ccusage) もあります。

| 機能                 | claude-tmux-status         | ccusage                      |
| -------------------- | -------------------------- | ---------------------------- |
| 表示場所             | tmuxステータスバー（常時） | ターミナル（要コマンド実行） |
| リアルタイム更新     | ✅                         | ❌                           |
| APIレート制限の表示  | ✅                         | ❌                           |
| 詳細な日別・月別集計 | ❌                         | ✅                           |
| インストール不要     | ❌                         | ✅（npx で即実行）           |

両者は用途が異なります。**常時監視したい**なら claude-tmux-status、**詳しく使用量を分析したい**なら ccusage、という使い分けが自然です。

組み合わせて使うのが最強です。

---
layout: post
---

## こんな方におすすめ

- **tmuxをメインで使っている開発者**：常時確認できるので使いすぎ防止に
- **Claude Code をヘビーに使っている方**：上限到達による作業中断を減らせます
- **コストを意識したい方**：コストモードで日々の出費をリアルタイムで把握

逆に、tmuxを使っていない場合はこのツールの恩恵を受けられません。その場合は ccusage が便利です。

---
layout: post
---

## 参考リンク

- [Claude Code の使用量上限とうまく付き合う方法](https://zenn.dev/long910/articles/2026-02-21-claude-code-usage-limit) — 使用量制限の仕組みを詳しく解説
- [ccusage - GitHub](https://github.com/ryoppippi/ccusage) — 詳細な使用量分析ツール
- [tmux 公式ドキュメント](https://github.com/tmux/tmux/wiki) — status-right の設定方法など
