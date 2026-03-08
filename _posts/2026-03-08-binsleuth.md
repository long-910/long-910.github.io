---
layout: post
title: "Rustで作ったバイナリ静的解析ツール「BinSleuth」の紹介"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["rust", "security", "binary", "cli", "oss"]
date: "2026-03-08 20:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-03-08-binsleuth)でも公開しています。


:::message
この記事は **Claude Code**（claude.ai/code）を使用して作成しました。
:::

## はじめに

セキュリティエンジニアや開発者として、こんな場面に遭遇したことはありませんか？

- ビルドした実行ファイルにセキュリティ強化オプションが有効になっているか確認したい
- 手に入れたバイナリが本当に安全かどうか、パッキングや暗号化の痕跡がないか調べたい
- CI/CDパイプラインに「ハードニングチェック」を組み込みたいが、重いツールは入れたくない

そんな課題を解決するために、Rustで **BinSleuth** を作りました。

https://github.com/long-910/BinSleuth

## BinSleuth とは

**BinSleuth**（ビン・スルース）は、ELFおよびPEバイナリに対してセキュリティ強化チェックとエントロピー解析を行うCLIツールです。

- **ゼロ依存** — システムライブラリ不要、純粋なRustで実装
- **高速** — ミリ秒オーダーで解析完了
- **CI対応** — `--strict` モードで終了コードによるパイプライン制御が可能
- **crates.io 公開済み** — `cargo install binsleuth` だけで導入完了

## インストール

```bash
cargo install binsleuth
```

Rust 1.85以降が必要です。それ以外の依存関係は一切ありません。

ソースからビルドする場合は：

```bash
git clone https://github.com/long-910/BinSleuth.git
cd BinSleuth
cargo build --release
```

## 主な機能

### 1. セキュリティ強化フラグのチェック

コンパイル済みバイナリに以下の保護機能が有効かどうかを検査します：

| フラグ | 説明 | ELF | PE |
|--------|------|-----|----|
| **NX** | スタック/データ領域の非実行化（コードインジェクション防止） | `PT_GNU_STACK` | `NX_COMPAT` |
| **PIE** | 位置独立実行ファイル（ASLR有効化） | `ET_DYN` | `DYNAMIC_BASE` |
| **RELRO** | 再配置テーブルの読み取り専用化（GOT上書き攻撃防止） | `PT_GNU_RELRO` + `BIND_NOW` | N/A |
| **Stack Canary** | バッファオーバーフロー検知シンボルの有無 | `__stack_chk_fail` | `__security_cookie` |
| **Stripped** | デバッグシンボルの有無 | `.debug_*` セクション | Debugディレクトリ |

各チェック結果は **Enabled** / **Partial** / **Disabled** / **N/A** のいずれかで表示されます。

### 2. セクションエントロピー解析

シャノンエントロピーの計算式：

```
H = -Σ P(x) · log₂(P(x))    範囲: [0.0 – 8.0]
```

| エントロピー範囲 | 判定 |
|------------------|------|
| 0.0 – 4.0 | 通常のコード／データ |
| 4.0 – 7.0 | 圧縮リソース（正常範囲） |
| **> 7.0** | **⚠ パッカー／暗号化の疑い** |

エントロピーが高いセクションは、UPXなどのパッカーで圧縮されたバイナリや、暗号化されたペイロードの存在を示唆します。マルウェア解析の初手として非常に有効です。

### 3. 危険シンボル検出

不審なバイナリによく登場する関数シンボルをフラグします：

| カテゴリ | 例 |
|----------|----|
| コード実行系 | `system`, `execve`, `popen`, `WinExec`, `CreateProcess` |
| ネットワーク系 | `connect`, `socket`, `gethostbyname`, `WinHttpOpen` |
| メモリ操作系 | `mprotect`, `mmap`, `VirtualAlloc`, `VirtualProtect` |

## 使い方

### 基本的な解析

```bash
# ELFバイナリ（Linux）
binsleuth /usr/bin/ls

# PEバイナリ（Windows）
binsleuth ./myapp.exe

# 不審なバイナリの調査
binsleuth ./suspicious_binary
```

### 全セクションを表示（低エントロピーも含む）

```bash
binsleuth --verbose /usr/bin/python3
```

### JSON出力（スクリプト連携・CI活用）

```bash
# NXフラグだけ取り出す
binsleuth --json /usr/bin/ls | jq '.hardening.nx'

# 解析結果をファイルに保存
binsleuth --json ./myapp > report.json
```

### CI/CDパイプラインへの組み込み

```bash
# ハードニング問題があれば終了コード2でフェイル
binsleuth --strict ./myapp && echo "Hardening OK" || echo "Hardening FAILED"
```

GitHubActionsでの活用例：

```yaml
- name: Binary hardening check
  run: binsleuth --strict ./target/release/myapp
```

## 実際の出力例

### 強化済みバイナリ（`/usr/bin/ls`）

```
╔══════════════════════════════════════════════════════╗
║              BinSleuth — Binary Analyzer             ║
╚══════════════════════════════════════════════════════╝

  File:    /usr/bin/ls
  Format:  ELF
  Arch:    X86_64

  ── Security Hardening ──────────────────────────────────

  [ ENABLED  ]  NX (Non-Executable Stack)
  [ ENABLED  ]  PIE (ASLR-compatible)
  [ ENABLED  ]  RELRO (Read-Only Relocations)
  [ ENABLED  ]  Stack Canary
  [ ENABLED  ]  Debug Symbols Stripped

  ── Section Entropy ─────────────────────────────────────

  All sections within normal entropy range.
  (run with --verbose to show all sections)

  ── Dangerous Symbol Usage ──────────────────────────────

  No dangerous symbols detected.

  Analysis complete.
```

### パック済みバイナリ（UPX圧縮）

```
  ── Section Entropy ─────────────────────────────────────

  Section          Size (B)    Entropy  Status
  ────────────────────────────────────────────────────────
  UPX0              491520      7.9981  ⚠  Packed/Encrypted suspected
  UPX1               32768      7.9912  ⚠  Packed/Encrypted suspected

  2 section(s) with entropy > 7.0 detected!

  ── Dangerous Symbol Usage ──────────────────────────────

  3 dangerous symbol(s) found:
    ▶  execve
    ▶  mprotect
    ▶  socket
```

## 対応フォーマット

| フォーマット | アーキテクチャ | NX | PIE | RELRO | Canary |
|--------------|---------------|----|-----|-------|--------|
| ELF 32-bit | x86, ARM, MIPS, … | ✅ | ✅ | ✅ | ✅ |
| ELF 64-bit | x86-64, AArch64, … | ✅ | ✅ | ✅ | ✅ |
| PE 32-bit (PE32) | x86 | ✅ | ✅ | N/A | ✅ |
| PE 64-bit (PE32+) | x86-64 | ✅ | ✅ | N/A | ✅ |

## 終了コードの設計

| コード | 意味 |
|--------|------|
| `0` | 解析成功・問題なし |
| `1` | ファイルが見つからない、パースエラー、非対応フォーマット |
| `2` | `--strict` モード：ハードニング問題を検出 |

`--strict` と終了コードの組み合わせにより、CI/CDとの統合が非常にシンプルになります。

## 実装のポイント

### なぜRustで作ったか

- **安全性** — バイナリ解析はメモリ境界違反が起きやすい処理。Rustの所有権モデルがそれを防いでくれる
- **速度** — Python/Ruby製ツールと比較してミリ秒オーダーで完了。CI実行時間を圧迫しない
- **ゼロ依存** — `libelf` や `libpe` などのシステムライブラリ不要。Dockerコンテナへの組み込みが容易

### 使用クレート

| クレート | 用途 |
|----------|------|
| [object](https://crates.io/crates/object) | ELF/PEバイナリのクロスプラットフォーム解析 |
| [clap](https://crates.io/crates/clap) | CLI引数パーサー |
| [anyhow](https://crates.io/crates/anyhow) | エルゴノミックなエラーハンドリング |
| [colored](https://crates.io/crates/colored) | ターミナルカラー出力 |

### テスト戦略

```bash
cargo test        # 全テスト実行
cargo test --lib  # ユニットテストのみ
cargo test --test cli  # 統合テストのみ
```

現在42テストが通過しています：

| モジュール | テスト数 | カバー内容 |
|------------|----------|-----------|
| `analyzer::entropy` | 9 | シャノン計算式、エッジケース、単調性 |
| `analyzer::hardening` | 13 | PEヘッダー解析、RELRO状態、ELF自己解析 |
| `tests::cli` | 20 | CLIフラグ、JSON出力、strictモード、エラー処理 |

## ユースケース

### セキュリティエンジニア向け

- マルウェア解析の初手として、パッキングの有無を即座に確認
- 脆弱性診断時に対象バイナリのハードニング状況をレポート

### 開発者向け

- CI/CDに組み込んでリリースビルドのセキュリティ品質を自動チェック
- ビルドフラグの変更が意図通りに反映されているか確認（`-fstack-protector-strong` など）

### CTFプレイヤー向け

- pwnableチャレンジで「どの緩和策が無効か」を即座に特定
- 問題バイナリの事前調査を高速化

## ロードマップ

現在計画・検討中の機能：

- [ ] **SARIF出力** — GitHub Security tab との統合
- [ ] **macOS Mach-O対応** — Apple Silicon含む
- [ ] **バイナリ差分** — `binsleuth diff a.out b.out` でインポートテーブルの変化を追跡
- [ ] **Yaraスタイルのパターンマッチング** — カスタムシグネチャによる検査

## 試してみよう

```bash
# インストール
cargo install binsleuth

# まず手元のシステムバイナリで試す
binsleuth /usr/bin/ls
binsleuth /usr/bin/ssh

# JSON出力で詳細確認
binsleuth --json /usr/bin/curl | jq .
```

フィードバック・Issue・PRは大歓迎です！

https://github.com/long-910/BinSleuth

## おわりに

BinSleuthはまだ若いプロジェクトですが、「重いリバースエンジニアリングツールを起動するほどでもない、でも素早くセキュリティ状態を把握したい」というニーズにしっかり応えられるツールを目指しています。

セキュリティの文脈でRustを使うことの安心感、そしてゼロ依存での配布のしやすさは、このツールの大きな強みです。ぜひ一度試してみてください。

---

*crates.io: https://crates.io/crates/binsleuth*
*docs.rs: https://docs.rs/binsleuth*
