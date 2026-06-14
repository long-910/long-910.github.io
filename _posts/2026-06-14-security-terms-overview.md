---
layout: post
title: "セキュリティ用語を体系的に整理する——ツール・攻撃手法・暗号技術を一気に押さえる"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "network", "pentesting", "crypto", "blockchain"]
date: "2026-06-14 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-14-security-terms-overview)でも公開しています。


## はじめに

セキュリティの学習を進めると、ツール名・攻撃手法名・プロトコル名が混在して出てきます。本記事では以下のキーワードをカテゴリごとに整理し、それぞれの概要・用途・背景をまとめました。

:::message alert
本記事の内容は教育・学習目的です。自分が管理・所有するシステム、または書面による明示的な許可を得たシステム以外には適用しないでください。
:::

---

## 1. ネットワーク解析・パケット操作ツール

### Traffic IQ Professional

**Traffic IQ Professional** は、ネットワークトラフィックの生成・解析・シミュレーションを行うツールです。

| 項目 | 内容 |
|------|------|
| 主な用途 | IDS/IPS の検証、ネットワーク負荷テスト |
| 特徴 | 実際の攻撃トラフィックを模したシナリオを再生できる |
| 対象者 | セキュリティエンジニア、ネットワーク管理者 |

IDS（侵入検知システム）がどのようなトラフィックに反応するかを事前に確認したいとき、実際の攻撃パターンを含むトラフィックを安全な環境で流すことができます。

---

### Colasoft Packet Builder

**Colasoft Packet Builder** は GUI でパケットを自由に組み立て送信できるツールです。

```
特徴
├── イーサネット/IP/TCP/UDP/HTTP など多層にわたるヘッダを手動編集
├── テンプレートパケットの読み込み・保存
└── 指定したレートでの連続送信
```

ファジング（不正な入力を送り込む手法）やプロトコルの動作確認に使われます。Wireshark でキャプチャしたパケットをインポートして再送することも可能です。

---

### ntpq / ntpdc

**ntpq** と **ntpdc** は NTP（Network Time Protocol）サーバーへの問い合わせツールです。

```bash
# NTP サーバーの状態確認
ntpq -p ntp.example.com

# ピアの一覧表示
ntpdc -c monlist ntp.example.com  # ※ 増幅攻撃に悪用される設定
```

| コマンド | 役割 |
|---------|------|
| `ntpq` | 標準的な NTP 状態照会 |
| `ntpdc` | NTP デーモン制御・詳細統計取得 |

**セキュリティ上の注意点**：`ntpdc -c monlist` は NTP 増幅 DDoS 攻撃に悪用されます。応答が要求の数百倍のサイズになるため、外部に公開した NTP サーバーでは `monlist` を無効化するのが鉄則です。

---

## 2. 脆弱性スキャナー

セキュリティ診断で使われる主要な脆弱性スキャナーを比較します。

| ツール | 種別 | 特徴 |
|--------|------|------|
| **OpenVAS** | OSS | Greenbone Vulnerability Management の中核エンジン。無償で使える本格スキャナー |
| **Qualys VM** | SaaS | クラウド型。エージェントレスでも動作。コンプライアンスレポート機能が充実 |
| **Nikto** | OSS | Web サーバー特化。Apache/Nginx の設定ミス・既知 CVE を素早くチェック |
| **Nessus** | 商用/無料版あり | 業界標準。プラグイン数が最多クラス。Tenable 社製 |

### OpenVAS の基本的な使い方

```bash
# Kali Linux でのセットアップ
sudo apt install openvas
sudo gvm-setup
sudo gvm-start

# ブラウザで https://127.0.0.1:9392 にアクセス
```

### Nikto の基本的な使い方

```bash
# Web サーバーをスキャン（自分のテスト環境のみ）
nikto -h http://localhost -p 80

# SSL スキャン
nikto -h https://localhost -ssl
```

---

## 3. PDF マルウェア解析ツール

悪意のある PDF ファイルの解析に特化したツールです。

### pdfid

**pdfid** は PDF ファイルの構造を素早く把握するための Python 製ツールです。

```bash
pip install pdfid
pdfid suspicious.pdf
```

出力例：
```
PDFiD 0.2.8 suspicious.pdf
 PDF Header: %PDF-1.6
 obj                   47
 endobj                47
 stream                10
 /JS                    1   ← JavaScript あり（要注意）
 /JavaScript            1   ← JavaScript あり（要注意）
 /OpenAction            1   ← 自動実行アクション
 /Launch                0
```

`/JS`、`/JavaScript`、`/OpenAction` などの存在がマルウェアの指標（IoC）になります。

---

### PDFStreamDumper

**PDFStreamDumper** は Windows 向けの GUI ツールで、PDF 内部のストリームを展開・デコードして詳細解析できます。

```
機能
├── 難読化されたストリームの自動デコード（Flate/LZW/ASCIIHex など）
├── 埋め込み JavaScript の抽出・実行トレース
├── シェルコードの検出とアセンブリ表示
└── 既知エクスプロイトパターンとの照合
```

`pdfid` でフラグが立った PDF を、PDFStreamDumper で深掘りする二段構えが一般的なワークフローです。

---

## 4. ネットワーク攻撃ツールと手法

### BetterCAP（ベターキャップ）

**BetterCAP** は Golang 製のモジュール式 MITM（中間者攻撃）フレームワークです。

```bash
# インストール（macOS）
brew install bettercap

# 対話型セッション開始
sudo bettercap -iface en0
```

```
bettercap > net.probe on          # ホスト探索
bettercap > net.show              # 発見したホスト一覧
bettercap > arp.spoof on          # ARP スプーフィング開始
bettercap > https.proxy on        # HTTPS プロキシ起動
```

主な機能：ARP スプーフィング、DNS スプーフィング、HTTP/HTTPS プロキシ、BLE スニッフィング、Wi-Fi パケットキャプチャ。

---

### Hetty（へてぃ）

**Hetty** は Go 製のオープンソース HTTP セキュリティリサーチツールキットです。Burp Suite の OSS 代替として開発されました。

| 機能 | 概要 |
|------|------|
| MITM プロキシ | HTTP/HTTPS トラフィックの傍受・改ざん |
| リクエスト履歴 | GraphQL API で過去のリクエストを検索・再送 |
| インターセプト | リクエスト・レスポンスを手動で編集してから転送 |

```bash
go install github.com/dstotijn/hetty/cmd/hetty@latest
hetty  # ブラウザで http://localhost:8080 が開く
```

---

### BPDUガード

**BPDUガード（BPDU Guard）** は攻撃への対策であり、スイッチの設定機能です。

STP（Spanning Tree Protocol）では、スイッチ間で BPDU（Bridge Protocol Data Unit）パケットを交換してトポロジーを決定します。攻撃者が不正なスイッチを接続し偽の BPDU を流すと、ネットワークトポロジーを操作できてしまいます（STP 攻撃）。

```
BPDUガードの動作
┌──────────────────────────────┐
│  エンドデバイス接続ポートで   │
│  BPDU を受信した瞬間に        │
│  → ポートを自動的にシャットダウン │
└──────────────────────────────┘
```

Cisco IOS での設定例：

```
interface FastEthernet0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
```

---

### Blind Hijacking（ブラインドハイジャッキング）

**Blind Hijacking** は TCP セッションハイジャッキングの一形態で、攻撃者が被害者とサーバー間の応答パケットを見られない状態でも、シーケンス番号を推測してセッションに割り込む手法です。

```
通常のセッションハイジャッキング：攻撃者がトラフィックを傍受できる
Blind Hijacking            ：応答が見えなくてもシーケンス番号を推測して注入
```

対策：TLS による通信の暗号化、ランダムな初期シーケンス番号（ISN）の使用、TCP MD5 認証。

---

### 断片化攻撃（Fragmentation Attack）

IP パケットはサイズが MTU（最大転送単位）を超えると**フラグメント（断片）** に分割されます。断片化攻撃はこの仕組みを悪用します。

```
攻撃の種類
├── オーバーラッピングフラグメント：断片を意図的に重ねて IDS を混乱させる
├── タイニーフラグメント      ：ヘッダを複数フラグメントに分割して検査を回避
└── フラグメントフラッド       ：不完全な断片を大量送信してバッファを枯渇させる
```

ファイアウォールやIDS は完全に再構成してから検査する必要があるため、負荷が高くなります。

---

### ティアドロップアタック（Teardrop Attack）

**ティアドロップアタック** は断片化攻撃の具体的な実装で、オフセットが互いに重なり合う不正な IP フラグメントを送り付けます。

```
正常なフラグメント：
  [  フラグメント1: offset=0,  len=200  ]
  [  フラグメント2: offset=200, len=200 ]

ティアドロップ（不正）：
  [  フラグメント1: offset=0,  len=200  ]
  [  フラグメント2: offset=100, len=200 ]  ← 重複！
```

再構成処理でオフセット計算が負の値になり、カーネルがクラッシュします。1990年代の Windows NT/95 で深刻な被害をもたらしました。現代の OS はパッチ済みですが、古い組み込み機器では今も脆弱なケースがあります。

---

### バトルジャッキング（Warjacking）

**バトルジャッキング（War Jacking）** はウォードライビング（wardriving）の延長で、車で走行しながら脆弱な Wi-Fi アクセスポイントを発見し、そのセッションをハイジャックする手法です。

```
ウォードライビング  → 脆弱 AP を発見・記録
バトルジャッキング → 発見した AP に接続してセッションを奪取
```

- WEP/WPA2-TKIP など旧来の暗号化方式を悪用
- 接続中ユーザーのセッション Cookie を窃取
- ARP スプーフィングと組み合わせて MITM を実現

対策：WPA3 の利用、公衆 Wi-Fi では VPN を必ず使用。

---

## 5. Web / API セキュリティ

### SOAP と SOAP インジェクション

**SOAP（Simple Object Access Protocol）** は XML ベースのメッセージプロトコルで、Web サービスの通信に広く使われてきました。

```xml
<!-- SOAP リクエスト例 -->
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
  <soapenv:Body>
    <GetUser>
      <userId>123</userId>
    </GetUser>
  </soapenv:Body>
</soapenv:Envelope>
```

**SOAP インジェクション**：XML 特殊文字（`<`, `>`, `&`, `'`, `"`）を入力に混入し、メッセージ構造を改ざんする攻撃です。

```xml
<!-- 悪意のある入力 userId: 123</userId><role>admin -->
<GetUser>
  <userId>123</userId><role>admin</role></userId>
</GetUser>
```

**XXE（XML 外部エンティティ）攻撃** も SOAP を経由して行われることがあります。対策は入力のエスケープと XXE 無効化です。

---

### ピギーバッククエリ（Piggyback Query）

**ピギーバッククエリ** は SQL インジェクションの一種で、正規のクエリに「便乗（piggyback）」して追加クエリを実行します。別名 **スタックドクエリ（Stacked Queries）** とも呼ばれます。

```sql
-- 正規クエリ
SELECT * FROM users WHERE id = '1'

-- ピギーバック攻撃（; で区切って追加クエリを注入）
SELECT * FROM users WHERE id = '1'; DROP TABLE users; --'
```

PostgreSQL や SQL Server では複数クエリを `;` で区切って実行できるため有効です。MySQL では PDO の `PDO::ATTR_EMULATE_PREPARES` を無効にするなどの設定が必要です。

**対策**：プリペアドステートメント（パラメータバインド）を徹底することで完全に防げます。

---

### CPDoS攻撃（Cache Poisoned Denial of Service）

**CPDoS（Cache Poisoned DoS）** は CDN や Web キャッシュサーバーを悪用した DoS 攻撃です。

```
攻撃フロー
攻撃者 ──[不正ヘッダ付きリクエスト]──→ CDN/キャッシュ
CDN   ──[オリジンサーバーへ転送]──→ オリジン
オリジン ─→ エラーレスポンス (400/403/404) を返す
CDN   ←──── エラーレスポンスをキャッシュ ←────
以降の正規ユーザー全員がキャッシュされたエラーを受け取る
```

主なバリアント：

| 手法 | 概要 |
|------|------|
| HHO（HTTP Header Oversize） | 巨大ヘッダでオリジンにエラーを返させる |
| HMC（HTTP Meta Character） | 制御文字を含むヘッダを送り込む |
| HTTP Method Override | `X-HTTP-Method-Override` ヘッダを悪用 |

**対策**：CDN でエラーレスポンスをキャッシュしない設定、`Cache-Control: no-store` の活用。

---

## 6. クラウド・インフラセキュリティ

### IMDS攻撃（Instance Metadata Service Attack）

**IMDS（Instance Metadata Service）** は AWS・GCP・Azure などのクラウド VM から自身のメタデータ（IAM ロール、認証トークンなど）を取得するための内部 HTTP エンドポイントです。

```
AWS: http://169.254.169.254/latest/meta-data/
GCP: http://metadata.google.internal/computeMetadata/v1/
```

**SSRF（Server-Side Request Forgery）との組み合わせ**が最も多い攻撃パターンです。

```
攻撃フロー（SSRF → IMDS）
攻撃者 ──[URL=http://169.254.169.254/...]──→ 脆弱な Web アプリ
Web アプリ ──[内部リクエスト]──→ IMDS
IMDS ──[IAM 一時クレデンシャル]──→ Web アプリ
Web アプリ ──[クレデンシャル漏洩]──→ 攻撃者
```

攻撃者は取得したクレデンシャルで AWS S3 や EC2 を操作できてしまいます。

**IMDSv2（対策）**：リクエストにセッショントークン（PUT リクエストで事前取得）を必須にすることで SSRF からの直接アクセスをブロックします。

```bash
# IMDSv2 でのトークン取得（SSRF では PUT できないためブロック可能）
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/
```

---

## 7. ブロックチェーン・DeFi 攻撃

### エクリプスアタック（Eclipse Attack）

**エクリプスアタック** は P2P ネットワークにおいて、特定のノードの全接続先を攻撃者が制御するノードで埋め尽くす攻撃です。

```
正常状態：ノード A ─── 正規ノード群（複数）
                  └─── 正規ノード群（複数）

エクリプス後：ノード A ─── 攻撃者ノード 1
                      ─── 攻撃者ノード 2
                      └─── 攻撃者ノード 3  ← 外界から遮断！
```

被害ノードは攻撃者が提供する偽のブロックチェーンデータしか受け取れなくなります。これを利用して二重支払い（ダブルスペンド）を誘発したり、51%攻撃のコストを下げたりします。

**対策**：接続ピアの多様化、ランダムなピア選択アルゴリズム。

---

### フィニーアタック（Finney Attack）

**フィニーアタック** はマイナー（採掘者）が行える二重支払い攻撃で、Hal Finney 氏にちなんで命名されています。

```
攻撃手順
1. 攻撃者（マイナー）がブロックを採掘し、自分自身への送金トランザクション X を含める
2. そのブロックを公開せず手元に保持
3. 商人に同じコインを送金するトランザクション Y を放送し、0-confirmation で支払い確認を受ける
4. 直後に手元のブロックを公開 → トランザクション X が先に記録され Y が無効化
```

**対策**：少額決済でも最低 1 ブロック（約 10 分）の承認を待つこと。

---

### DeFi サンドイッチ攻撃（Sandwich Attack）

**サンドイッチ攻撃** は DeFi の AMM（自動マーケットメーカー）を標的にした MEV（Maximal Extractable Value）攻撃です。

```
攻撃フロー（ユーザーが ETH → USDC のスワップを行う場合）
1. 攻撃者：ユーザーのトランザクションを mempool で検出
2. 攻撃者：高いガス代でユーザーより先に ETH → USDC を買う（フロントラン）
   → プールの価格が上昇
3. ユーザー：高くなった価格で ETH → USDC をスワップ（損失）
4. 攻撃者：ユーザーのトランザクション後に USDC → ETH を売る（バックラン）
   → 差益を確定
```

攻撃者はユーザーのトランザクションを「パン（攻撃者の2つのトランザクション）」で挟みます。

**対策**：スリッページ許容値を低く設定する、MEV-Blocker RPC を使用する、Flashbots Protect を利用する。

---

## 8. 暗号技術

### Blowfish

**Blowfish** は 1993 年に Bruce Schneier が設計した対称鍵ブロック暗号です。

| 項目 | 詳細 |
|------|------|
| ブロックサイズ | 64 ビット |
| 鍵長 | 32〜448 ビット（可変） |
| ラウンド数 | 16 ラウンドの Feistel 構造 |
| 特徴 | 鍵スケジュールが重く、ブルートフォースに強い |
| 用途 | bcrypt のコアアルゴリズム（パスワードハッシュ） |

```python
from Crypto.Cipher import Blowfish
from Crypto.Util.Padding import pad

key = b'SecretKey'
cipher = Blowfish.new(key, Blowfish.MODE_CBC)
ct = cipher.encrypt(pad(b'Hello, World!', Blowfish.block_size))
```

ブロックサイズが 64 ビットのため、大量データの暗号化では **SWEET32 攻撃**（生日攻撃）のリスクがあります。新規設計では AES や Twofish を推奨します。

---

### Twofish

**Twofish** は 1998 年に Schneier らが開発し、AES 選定コンペのファイナリストになったブロック暗号です。

| 項目 | 詳細 |
|------|------|
| ブロックサイズ | 128 ビット |
| 鍵長 | 128/192/256 ビット |
| 構造 | Feistel ネットワーク（16 ラウンド） |
| 特徴 | 鍵依存の S-Box、MDS 行列による拡散 |
| 現在の位置づけ | 安全性は高いが AES の普及で採用例は減少 |

Blowfish の後継として設計されており、64 ビットブロックの制約を解消しています。AES が選ばれた理由は安全性の差ではなく、ハードウェア実装の効率性でした。

---

## カテゴリ別まとめ

### ツール系

| ツール | カテゴリ | 主な用途 |
|--------|---------|---------|
| Traffic IQ Professional | ネットワーク | トラフィック生成・IDS 検証 |
| Colasoft Packet Builder | ネットワーク | パケット手動生成・送信 |
| BetterCAP | MITM | ARP/DNS スプーフィング |
| Hetty | Web | HTTP プロキシ・インターセプト |
| OpenVAS | 脆弱性スキャン | ネットワーク全般の脆弱性検出 |
| Qualys VM | 脆弱性スキャン | クラウド型・コンプライアンス |
| Nikto | 脆弱性スキャン | Web サーバー専用 |
| Nessus | 脆弱性スキャン | 汎用・プラグイン最多 |
| pdfid | マルウェア解析 | PDF 構造の素早い把握 |
| PDFStreamDumper | マルウェア解析 | PDF の詳細解析・JavaScript 抽出 |
| ntpq / ntpdc | NTP | NTP サーバー診断・監視 |

### 攻撃手法系

| 手法 | 層 | 概要 |
|------|---|------|
| 断片化攻撃 | ネットワーク層 | IP フラグメントの不正操作 |
| ティアドロップ | ネットワーク層 | 重複フラグメントでカーネルクラッシュ |
| Blind Hijacking | トランスポート層 | シーケンス番号推測による TCP ハイジャック |
| バトルジャッキング | データリンク/ネットワーク | Wi-Fi セッションの移動式ハイジャック |
| ピギーバッククエリ | アプリケーション層 | セミコロンによるクエリ便乗 |
| SOAP インジェクション | アプリケーション層 | XML 構造の改ざん |
| CPDoS | CDN/キャッシュ | キャッシュにエラーを混入 |
| IMDS 攻撃 | クラウドインフラ | SSRF 経由でメタデータを盗取 |
| エクリプス | P2P/ブロックチェーン | ノードの通信を全て制御 |
| フィニーアタック | ブロックチェーン | マイナーによる二重支払い |
| サンドイッチ攻撃 | DeFi | フロント/バックランによる差益搾取 |

### 防御設定系

| 機能 | 効果 |
|------|------|
| BPDUガード | 不正スイッチによる STP 操作を防ぐ |
| IMDSv2 | SSRF からメタデータを保護 |

### 暗号アルゴリズム

| アルゴリズム | 安全性 | 推奨 |
|------------|--------|------|
| Blowfish | 64 ビットブロックのため SWEET32 リスク | 新規では非推奨（bcrypt での用途は除く）|
| Twofish | 高い | AES の代替として引き続き有効 |

---

## おわりに

本記事で取り上げたキーワードは、ネットワーク・Web・クラウド・ブロックチェーンと幅広い領域に分散しています。「どの層で起きる問題か」を意識することが体系的な理解の近道です。

セキュリティの学習では、これらのツールを**自分が管理する環境（ローカル VM、CTF プラットフォーム、許可を得たラボ環境）**でのみ試してみることを強くお勧めします。
