---
layout: post
title: "MacBook M5 に John the Ripper をインストールして使ってみた"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "mac", "johntheripper", "password", "homebrew"]
date: "2026-06-03 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-03-john-the-ripper-mac-m5)でも公開しています。


## はじめに

セキュリティ勉強の一環として、**John the Ripper（JtR）** というパスワードクラッキングツールを調べてみました。「クラッキング」という言葉が物騒に聞こえますが、実際には**自分が忘れたパスワードの回復**や**パスワードポリシーの強度テスト**など、防御的なセキュリティ学習にも広く使われています。

本記事では MacBook M5（Apple Silicon）への導入から基本的な使い方まで、自分が調べた内容をまとめます。

:::message alert
**注意**: John the Ripper は自分が管理するシステム・ファイルに対してのみ使用してください。第三者のアカウントやシステムに無断で使用することは**不正アクセス禁止法などの法律に違反する**可能性があります。本記事の内容は教育目的のみを想定しています。
:::

---

## John the Ripper とは

**John the Ripper**（通称 JtR / John）は、1990 年代から開発されている老舗のオープンソース パスワードクラッカーです。

| 項目 | 内容 |
|------|------|
| 開発元 | Openwall Project |
| ライセンス | GPL（コミュニティ版） |
| 対応 OS | Linux / macOS / Windows |
| 対応ハッシュ | MD5, SHA-1, SHA-256, bcrypt, NTLM など数百種類 |

主に以下の用途で使われます。

- 自分のパスワードファイルの強度チェック
- CTF（Capture The Flag）競技
- ペネトレーションテスト（許可を得た環境）
- パスワードポリシーの研究・学習

---

## バリアント：Jumbo vs Community

John the Ripper には 2 種類があります。

| バリアント | 特徴 |
|-----------|------|
| **Community Edition** | 公式シンプル版。対応ハッシュが少なめ |
| **Jumbo（拡張版）** | コミュニティが拡張。対応ハッシュが非常に多い。実用上はこちら推奨 |

Homebrew でインストールすると Jumbo 版が入ります。

---

## インストール方法（MacBook M5）

### 前提条件

- macOS Sequoia（または Ventura / Sonoma）
- Homebrew がインストール済み

Homebrew が未導入の場合は以下でインストールします。

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Homebrew でインストール

```bash
brew install john-jumbo
```

M5（Apple Silicon）でも問題なく動作します。インストール後のバイナリは `/opt/homebrew/bin/john` に配置されます。

インストール確認：

```bash
john --version
```

出力例：

```
John the Ripper 1.9.0-jumbo-1 [darwin-arm64]
```

---

## 基本的な使い方

### ハッシュファイルの準備

John はパスワードの**ハッシュ値**（暗号化された文字列）を入力として受け取ります。まず練習用のハッシュを用意しましょう。

```bash
# MD5 ハッシュを生成（"hello" のハッシュ）
echo -n "hello" | md5
# 出力: 5d41402abc4b2a76b9719d911017c592
```

以下の内容でテスト用ファイルを作成します。

```
# hash_test.txt
testuser:5d41402abc4b2a76b9719d911017c592
```

---

### モード 1: 単語リスト攻撃（Wordlist Attack）

あらかじめ用意した単語リストと照合する最も基本的な方法です。

```bash
john --wordlist=/usr/share/dict/words hash_test.txt
```

macOS には `/usr/share/dict/words` が標準で存在します（約 235,000 語）。

**より強力な単語リストを使う場合：**

有名な `rockyou.txt`（1,400 万件のリストを収録）を使うことができます。

```bash
# rockyou.txt は Kali Linux に同梱されているが、
# macOS では別途入手が必要（SecLists 等から取得可能）
john --wordlist=rockyou.txt hash_test.txt
```

---

### モード 2: ブルートフォース攻撃（Incremental Mode）

総当たりで全パターンを試す方法です。時間がかかりますが確実です。

```bash
john --incremental hash_test.txt
```

文字セットを指定することも可能です。

```bash
# 数字のみ（PIN コードなど）
john --incremental=Digits hash_test.txt

# アルファベット小文字のみ
john --incremental=Lower hash_test.txt

# アルファベット（大小）+ 数字
john --incremental=Alnum hash_test.txt
```

---

### モード 3: ルール攻撃（Rules Attack）

単語リストに変形ルールを適用する方法です。`hello` → `Hello`, `H3ll0`, `hello123` のような変形を自動生成します。

```bash
john --wordlist=/usr/share/dict/words --rules hash_test.txt
```

---

### モード 4: シングルクラックモード

ユーザー名やファイル内のメタ情報をもとにパスワードを推測します。

```bash
john --single hash_test.txt
```

---

## 結果の確認

クラックが成功したパスワードを確認するには：

```bash
john --show hash_test.txt
```

出力例：

```
testuser:hello

1 password hash cracked, 0 left
```

---

## `/etc/shadow` 形式のハッシュを扱う（Linux）

Linux の `/etc/shadow` ファイルを扱う場合、`unshadow` コマンドで `/etc/passwd` と結合します。

```bash
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt
```

> macOS 自体は `/etc/shadow` を使わないため、主に Linux 環境を扱うときの手順です。

---

## ZIP / RAR / PDF ファイルのパスワード解析

JtR Jumbo 版では圧縮ファイルやドキュメントのパスワード解析も可能です。専用のコンバーターでハッシュを抽出してから解析します。

### ZIP ファイル

```bash
# ハッシュ抽出
zip2john protected.zip > zip_hash.txt

# 解析
john zip_hash.txt
```

### PDF ファイル

```bash
# ハッシュ抽出
pdf2john protected.pdf > pdf_hash.txt

# 解析
john pdf_hash.txt
```

### SSH 秘密鍵（パスフレーズ忘れ時）

```bash
# ハッシュ抽出
ssh2john id_rsa > ssh_hash.txt

# 解析
john ssh_hash.txt
```

---

## コマンド・オプション一覧

### インストール・セットアップ

| コマンド | 説明 |
|---------|------|
| `brew install john-jumbo` | Jumbo 版をインストール |
| `john --version` | バージョン確認 |
| `john --list=formats` | 対応ハッシュ形式を全件表示 |
| `john --list=subformats` | サブフォーマット一覧表示 |

---

### ハッシュ抽出ツール（コンバーター）

| コマンド | 対象 |
|---------|------|
| `zip2john <file.zip> > hash.txt` | ZIP ファイル |
| `rar2john <file.rar> > hash.txt` | RAR ファイル |
| `pdf2john <file.pdf> > hash.txt` | PDF ファイル |
| `ssh2john <id_rsa> > hash.txt` | SSH 秘密鍵 |
| `gpg2john <key.gpg> > hash.txt` | GPG 秘密鍵 |
| `keepass2john <db.kdbx> > hash.txt` | KeePass データベース |
| `office2john <file.docx> > hash.txt` | Office ファイル（Word / Excel 等） |
| `unshadow /etc/passwd /etc/shadow > hash.txt` | Linux shadow ファイル結合 |

---

### クラックモード

| コマンド | モード | 説明 |
|---------|-------|------|
| `john hash.txt` | 自動 | モードを自動選択（single → wordlist → incremental の順） |
| `john --single hash.txt` | シングル | ユーザー名・メタ情報から推測 |
| `john --wordlist=<list> hash.txt` | 単語リスト | 辞書と照合 |
| `john --wordlist=<list> --rules hash.txt` | ルール | 辞書 ＋ 変形ルール適用 |
| `john --incremental hash.txt` | ブルートフォース | 全パターン総当たり |
| `john --incremental=Digits hash.txt` | ブルートフォース | 数字のみ |
| `john --incremental=Lower hash.txt` | ブルートフォース | 小文字のみ |
| `john --incremental=Alpha hash.txt` | ブルートフォース | 大文字 ＋ 小文字 |
| `john --incremental=Alnum hash.txt` | ブルートフォース | 英数字 |
| `john --mask=?l?l?l?d hash.txt` | マスク | パターン指定（`?l`=小文字, `?d`=数字） |

---

### ハッシュ形式の指定

| コマンド | 説明 |
|---------|------|
| `john --format=md5crypt hash.txt` | MD5crypt（Linux MD5）を指定 |
| `john --format=bcrypt hash.txt` | bcrypt を指定 |
| `john --format=sha256crypt hash.txt` | SHA-256crypt を指定 |
| `john --format=NT hash.txt` | Windows NTLM を指定 |
| `john --format=Raw-MD5 hash.txt` | Raw MD5（ソルトなし）を指定 |
| `john --format=Raw-SHA1 hash.txt` | Raw SHA-1 を指定 |

形式が不明な場合は `--list=formats` で対応形式を確認して照合します。

---

### セッション管理・一時停止・再開

| コマンド | 説明 |
|---------|------|
| `john --session=mysession hash.txt` | セッション名を付けて実行 |
| `john --restore=mysession` | 途中から再開 |
| `john --restore` | 直近のセッションを再開 |
| `Ctrl + C` | 実行を一時停止（セッションは自動保存） |

---

### 結果確認・出力

| コマンド | 説明 |
|---------|------|
| `john --show hash.txt` | クラック済みパスワードを表示 |
| `john --show --format=<形式> hash.txt` | 形式を指定して表示 |
| `john --pot=john.pot --show hash.txt` | pot ファイルを指定して表示 |
| `john --show=left hash.txt` | 未クラックのハッシュのみ表示 |

クラック結果は `~/.john/john.pot` に自動保存されます。

---

### パフォーマンス・並列化

| コマンド | 説明 |
|---------|------|
| `john --fork=<N> hash.txt` | N プロセスで並列実行 |
| `sysctl -n hw.logicalcpu` | 論理コア数を確認（fork 数の参考に） |
| `john --node=1/2 hash.txt` | 複数マシン分散（ノード 1/2 を担当） |
| `john --node=2/2 hash.txt` | 複数マシン分散（ノード 2/2 を担当） |

---

### マスクモードの文字セット記号

マスクモード（`--mask`）で使用できる記号：

| 記号 | 意味 |
|-----|------|
| `?l` | 小文字アルファベット（a–z） |
| `?u` | 大文字アルファベット（A–Z） |
| `?d` | 数字（0–9） |
| `?s` | 記号（!@#$ 等） |
| `?a` | 全印字可能文字（`?l` ＋ `?u` ＋ `?d` ＋ `?s`） |
| `?w` | 単語リストの単語 1 文字 |

例：8 文字の英数字 ＋ 末尾 1 桁の数字を試す

```bash
john --mask=?a?a?a?a?a?a?a?d hash.txt
```

---

## M5（Apple Silicon）での注意点

M5 は ARM アーキテクチャのため、以下の点に注意が必要でした。

### GPU アクセラレーションについて

JtR の Jumbo 版は OpenCL / CUDA を使った GPU アクセラレーションに対応していますが、**macOS の Metal GPU には対応していません**。M5 の GPU を活用したい場合は Hashcat（別ツール）を検討するとよいでしょう。

### パフォーマンス

CPU のみでも M5 の高いコア性能のおかげで十分な速度が出ます。`--fork` オプションで効率コア・性能コア両方をフル活用できます。

```bash
# 論理コア数を確認
sysctl -n hw.logicalcpu

# 8 コアで並列実行
john --fork=8 --wordlist=rockyou.txt hash_test.txt
```

---

## まとめ

| やること | コマンド |
|---------|---------|
| インストール | `brew install john-jumbo` |
| 単語リスト攻撃 | `john --wordlist=<list> <hash_file>` |
| ブルートフォース | `john --incremental <hash_file>` |
| ルール適用 | `john --wordlist=<list> --rules <hash_file>` |
| 結果確認 | `john --show <hash_file>` |
| ZIP 解析 | `zip2john <zip> > hash.txt && john hash.txt` |

John the Ripper はシンプルながら非常に強力なツールです。CTF やセキュリティ学習の環境で積極的に使ってみて、パスワードの脆弱性について理解を深めるきっかけになれば幸いです。

---

## 参考リンク

- [John the Ripper 公式サイト](https://www.openwall.com/john/)
- [Jumbo GitHub リポジトリ](https://github.com/openwall/john)
- [Homebrew: john-jumbo](https://formulae.brew.sh/formula/john-jumbo)
