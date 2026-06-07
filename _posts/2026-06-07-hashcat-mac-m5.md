---
layout: post
title: "MacBook M5 に Hashcat をインストールして使ってみた"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "mac", "hashcat", "password", "homebrew"]
date: "2026-06-07 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-07-hashcat-mac-m5)でも公開しています。


## はじめに

前回の記事で **John the Ripper** を試しましたが、John the Ripper は macOS の Metal GPU を活用できないという制限がありました。そこで今回は、**GPU アクセラレーション**に対応した高速パスワードクラッカー **Hashcat** を MacBook M5 で試してみました。

Hashcat は Apple Silicon の Metal GPU を活用できるため、CPU のみで動作する John the Ripper と比べて大幅に高速なクラッキングが期待できます。

:::message alert
**注意**: Hashcat は自分が管理するシステム・ファイルに対してのみ使用してください。第三者のアカウントやシステムに無断で使用することは**不正アクセス禁止法などの法律に違反する**可能性があります。本記事の内容は教育目的のみを想定しています。
:::

---

## Hashcat とは

**Hashcat** は世界で最も広く使われているオープンソースのパスワードリカバリツールです。最大の特徴は **GPU アクセラレーション**に対応しており、CPU のみで動作するツールと比較して数十〜数百倍の速度でハッシュを解析できます。

| 項目 | 内容 |
|------|------|
| 開発元 | atom (Jens Steube) |
| ライセンス | MIT |
| 対応 OS | Linux / macOS / Windows |
| 対応ハッシュ | MD5, SHA-1, SHA-256, bcrypt, NTLM, WPA/WPA2 など 300 種類以上 |
| GPU 対応 | OpenCL / CUDA / Metal (Apple Silicon) |

主な用途：

- 自分のパスワードファイルの強度チェック
- CTF（Capture The Flag）競技
- ペネトレーションテスト（許可を得た環境）
- Wi-Fi（WPA/WPA2）パスワードの回復
- パスワードポリシーの研究・学習

---

## John the Ripper との比較

| 比較項目 | Hashcat | John the Ripper |
|---------|---------|-----------------|
| GPU アクセラレーション | ◎ 対応（Metal / CUDA / OpenCL） | △ macOS の Metal は非対応 |
| Apple Silicon (M5) GPU 活用 | ◎ 可能 | ✗ CPU のみ |
| 自動ハッシュ検出 | △ 手動指定が基本 | ◎ 自動検出 |
| ファイル形式サポート | △ ハッシュ抽出は別途ツールが必要 | ◎ `zip2john` 等が同梱 |
| 対応ハッシュ数 | ◎ 300 種類以上 | ◎ 数百種類 |
| 処理速度 | ◎ GPU 使用時は圧倒的に速い | ○ CPU のみ |

**結論**: 速度重視なら Hashcat、手軽さ・自動化重視なら John the Ripper という使い分けが一般的です。

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
brew install hashcat
```

インストール確認：

```bash
hashcat --version
```

出力例：

```
v6.2.6
```

### 利用可能なデバイスの確認

インストール後、まず利用可能な計算デバイスを確認します。

```bash
hashcat -I
```

出力例（M5 の場合）：

```
hashcat (v6.2.6) starting in backend information mode

Apple Metal API (Metal)
=======================
* Device #1: Apple M5, 21888/21845 MB, 16MCU

OpenCL API (OpenCL) - Platform #1 [Apple]
==========================================
* Device #2: Apple M5, skipped

* Device #3: Apple M5, 21888/21845 MB, 16MCU
```

GPU（Metal デバイス）が認識されていれば GPU アクセラレーションが利用できます。

---

## アタックモード

Hashcat には複数のアタックモードがあり、`-a` オプションで指定します。

| モード番号 | モード名 | 説明 |
|-----------|---------|------|
| `-a 0` | Straight（辞書攻撃） | 単語リストと照合 |
| `-a 1` | Combination（組み合わせ） | 2 つの単語リストを組み合わせ |
| `-a 3` | Brute-force（マスク攻撃） | パターンを指定した総当たり |
| `-a 6` | Hybrid wordlist + mask | 単語リスト ＋ マスク |
| `-a 7` | Hybrid mask + wordlist | マスク ＋ 単語リスト |
| `-a 9` | Association | ユーザー名などのヒントから推測 |

---

## 基本的な使い方

### ハッシュファイルの準備

Hashcat はパスワードの**ハッシュ値**を入力として受け取ります。まず練習用のハッシュを用意しましょう。

```bash
# MD5 ハッシュを生成（"hello" のハッシュ）
echo -n "hello" | md5
# 出力: 5d41402abc4b2a76b9719d911017c592
```

以下の内容でテスト用ファイルを作成します。

```
# hash_test.txt
5d41402abc4b2a76b9719d911017c592
```

> John the Ripper と異なり、Hashcat は `ユーザー名:ハッシュ` 形式ではなく**ハッシュのみ**を記述します（形式によって異なる場合もあります）。

---

### モード 1: 辞書攻撃（Straight Attack）

あらかじめ用意した単語リストと照合する最も基本的な方法です。

```bash
hashcat -m 0 -a 0 hash_test.txt /usr/share/dict/words
```

- `-m 0`: ハッシュタイプ（0 = MD5）
- `-a 0`: アタックモード（0 = Straight）

**より強力な単語リストを使う場合：**

```bash
hashcat -m 0 -a 0 hash_test.txt rockyou.txt
```

---

### モード 2: マスク攻撃（Brute-force Attack）

パターンを指定した総当たりです。`?l`, `?d` などの記号でパターンを定義します。

```bash
# 5 文字の小文字アルファベットを総当たり
hashcat -m 0 -a 3 hash_test.txt ?l?l?l?l?l
```

| 記号 | 意味 |
|-----|------|
| `?l` | 小文字アルファベット（a–z） |
| `?u` | 大文字アルファベット（A–Z） |
| `?d` | 数字（0–9） |
| `?s` | 記号（!@#$ 等） |
| `?a` | 全印字可能文字（`?l` ＋ `?u` ＋ `?d` ＋ `?s`） |
| `?b` | 0x00–0xFF（全バイト） |

例：8 文字の英数字（小文字 ＋ 数字）を総当たり

```bash
hashcat -m 0 -a 3 hash_test.txt ?a?a?a?a?a?a?a?a
```

---

### モード 3: ルール攻撃（Rule-based Attack）

単語リストにルールを適用して変形させます。`hello` → `Hello`, `H3ll0`, `hello123` のような変形を自動生成します。

```bash
hashcat -m 0 -a 0 hash_test.txt rockyou.txt -r /opt/homebrew/share/hashcat/rules/best64.rule
```

組み込みのルールファイルを確認する：

```bash
ls /opt/homebrew/share/hashcat/rules/
```

代表的なルールファイル：

| ルールファイル | 説明 |
|-------------|------|
| `best64.rule` | 汎用的な 64 種のルール |
| `rockyou-30000.rule` | rockyou ベースの強力なルール |
| `leetspeak.rule` | leet speak 変換（a→4, e→3 等） |
| `toggles5.rule` | 大文字・小文字のトグル |
| `dive.rule` | 非常に多くのルールを含む高強度ルール |

---

### モード 4: 組み合わせ攻撃（Combination Attack）

2 つの単語リストを結合してパスワードを試します（例: "pass" + "word" → "password"）。

```bash
hashcat -m 0 -a 1 hash_test.txt wordlist1.txt wordlist2.txt
```

---

## 結果の確認

クラック成功後、結果を確認するには：

```bash
hashcat -m 0 hash_test.txt --show
```

出力例：

```
5d41402abc4b2a76b9719d911017c592:hello
```

クラック結果は `~/.hashcat/hashcat.potfile` に自動保存されます。

---

## 主要なハッシュタイプ一覧

Hashcat ではハッシュタイプを `-m` オプションで数値指定します。

### 汎用ハッシュ

| `-m` 番号 | ハッシュタイプ |
|---------|------------|
| `0` | MD5 |
| `100` | SHA-1 |
| `1400` | SHA-256 |
| `1700` | SHA-512 |
| `3200` | bcrypt（$2*$）|
| `500` | MD5crypt（Unix MD5）|
| `1800` | sha512crypt（Unix SHA-512）|

### Windows 系

| `-m` 番号 | ハッシュタイプ |
|---------|------------|
| `1000` | NTLM |
| `3000` | LM |
| `5500` | NetNTLMv1 |
| `5600` | NetNTLMv2 |

### ネットワーク・Wi-Fi

| `-m` 番号 | ハッシュタイプ |
|---------|------------|
| `22000` | WPA-PBKDF2-PMKID+EAPOL（WPA/WPA2） |
| `2500` | WPA/WPA2（旧形式）|

### アプリケーション

| `-m` 番号 | ハッシュタイプ |
|---------|------------|
| `13400` | KeePass 1/2 |
| `17200` | PKZIP |
| `13600` | WinZip |
| `9600` | MS Office 2013 |

全ハッシュタイプの一覧は以下で確認できます。

```bash
hashcat --help | grep -E "^\s+[0-9]"
```

---

## コマンド・オプション一覧

### 基本オプション

| オプション | 説明 |
|---------|------|
| `-m <番号>` | ハッシュタイプを指定 |
| `-a <番号>` | アタックモードを指定 |
| `-o <ファイル>` | クラック結果をファイルに出力 |
| `--show` | クラック済みハッシュを表示 |
| `--left` | 未クラックのハッシュを表示 |
| `-r <ルール>` | ルールファイルを指定 |
| `--status` | 実行中のステータスをリアルタイム表示 |
| `--status-timer=<秒>` | ステータス更新間隔を設定 |

---

### セッション管理

| オプション | 説明 |
|---------|------|
| `--session=<名前>` | セッション名を付けて実行 |
| `--restore` | 直近のセッションを再開 |
| `--restore-file-path=<パス>` | 指定セッションファイルから再開 |
| `p` | 実行中に `p` キーでポーズ（一時停止） |
| `r` | 一時停止中に `r` キーで再開 |
| `q` | 実行中に `q` キーで終了（セッション保存） |

---

### デバイス・パフォーマンス

| オプション | 説明 |
|---------|------|
| `-I` | 利用可能デバイスを一覧表示 |
| `-D <番号>` | デバイスタイプ（1=CPU, 2=GPU） |
| `-d <番号>` | 使用するデバイス番号を指定 |
| `--force` | 警告を無視して強制実行（非推奨） |
| `-w <1-4>` | ワークロードプロファイル（1=省電力 〜 4=最大） |
| `-O` | カーネル最適化を有効化（速度向上） |

---

### ベンチマーク

各ハッシュタイプの処理速度を測定します。

```bash
# 全ハッシュタイプのベンチマーク
hashcat -b

# MD5 のみ
hashcat -b -m 0
```

---

## M5（Apple Silicon）での GPU アクセラレーション

M5 の大きな強みは **Metal GPU** を Hashcat で活用できる点です。John the Ripper とは異なり、Hashcat は Apple Silicon の GPU をフル活用できます。

### GPU 使用の確認

```bash
# Metal デバイスを使用（デフォルト）
hashcat -m 0 -a 0 hash_test.txt /usr/share/dict/words

# CPU のみ使用（比較用）
hashcat -m 0 -a 0 hash_test.txt /usr/share/dict/words -D 1
```

### ベンチマーク比較例（M5 Pro の場合）

```bash
hashcat -b -m 0
```

```
* Device #1: Apple M5 Pro, Metal
  Speed.#1.........: 8000 MH/s（MD5）
```

MD5 で約 80 億ハッシュ/秒という処理速度は、CPU のみの場合と比較して数十倍の高速化です。

### ワークロード設定

M5 でのパフォーマンスを最大化するには：

```bash
# ワークロード最大 + カーネル最適化
hashcat -m 0 -a 0 hash_test.txt rockyou.txt -w 4 -O
```

> `-w 4` はシステム全体のリソースを使い切るため、実行中は他の作業が重くなります。日常使用中は `-w 2` 程度が無難です。

---

## 実践例：ZIP ファイルのパスワード解析

Hashcat 自体はハッシュ抽出機能を持たないため、`hashcat-utils` や他のツールでハッシュを抽出してから使用します。

### ZIP ファイル

```bash
# zip2john（John the Ripper 付属）でハッシュを抽出
zip2john protected.zip > zip_hash.txt

# Hashcat 用の形式に変換（zip2john の出力をそのまま使う場合）
hashcat -m 17220 zip_hash.txt rockyou.txt
```

### WPA/WPA2 ハンドシェイク

Wi-Fi のパスワード解析（自分の Wi-Fi のみ）：

```bash
# hcxtools でキャプチャファイルを変換
hcxpcapngtool -o hash.hc22000 capture.pcapng

# 解析
hashcat -m 22000 hash.hc22000 rockyou.txt
```

---

## まとめ

| やること | コマンド |
|---------|---------|
| インストール | `brew install hashcat` |
| デバイス確認 | `hashcat -I` |
| 辞書攻撃 | `hashcat -m 0 -a 0 <hash> <wordlist>` |
| マスク攻撃 | `hashcat -m 0 -a 3 <hash> ?a?a?a?a?a` |
| ルール攻撃 | `hashcat -m 0 -a 0 <hash> <wordlist> -r best64.rule` |
| 結果確認 | `hashcat -m 0 <hash> --show` |
| ベンチマーク | `hashcat -b -m 0` |

Hashcat は Apple Silicon の GPU を活用できる点が John the Ripper との大きな違いです。処理速度が重要な場面（大量のハッシュ解析、WPA2 解析など）では Hashcat を選ぶのが賢明です。セキュリティ学習の一環として、自分の環境で安全に試してみてください。

---

## 参考リンク

- [Hashcat 公式サイト](https://hashcat.net/hashcat/)
- [Hashcat GitHub リポジトリ](https://github.com/hashcat/hashcat)
- [Hashcat Wiki](https://hashcat.net/wiki/)
- [Homebrew: hashcat](https://formulae.brew.sh/formula/hashcat)
- [Hashcat ハッシュタイプ一覧](https://hashcat.net/wiki/doku.php?id=hashcat)
