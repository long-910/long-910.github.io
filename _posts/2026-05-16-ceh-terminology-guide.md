---
layout: post
title: "レッドチームセキュリティ：攻撃フェーズ別に学ぶ重要用語ガイド"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "redteam", "pentest", "network", "ethical-hacking"]
date: "2026-05-26 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-16-ceh-terminology-guide)でも公開しています。


## はじめに

セキュリティエンジニアがより強固な防御を設計するには、**攻撃者（レッドチーム）の視点**を体系的に理解することが不可欠です。攻撃の各フェーズ・手法・ツールを把握することで、脅威モデリングや防御策の選択に深みが増します。

この記事は、ペネトレーションテストの標準的なフレームワークに沿って**セキュリティの重要用語を分野別に構造化した学習ノート**です。実務知識のリファレンスとしてご活用ください。

:::message alert
**重要：悪用禁止**

本記事は**セキュリティ学習・研究目的のまとめ**です。記載の手法・ツール・コードを**許可なく他者のシステムに使用することは不正アクセス禁止法等に違反する犯罪行為**です。

- 使用は必ず**自分が管理・所有する環境**、または**明示的な許可を得たペネトレーションテスト環境・CTF**に限定してください
- **絶対に悪用しないでください**
:::

---

## 攻撃の5フェーズ

ペネトレーションテストおよびレッドチーム演習では、以下の5フェーズが基本的な構造として使われます。

```
[1] Reconnaissance（偵察）
        ↓
[2] Scanning（スキャン）
        ↓
[3] Gaining Access（侵入）
        ↓
[4] Maintaining Access（維持）
        ↓
[5] Clearing Tracks（痕跡消去）
```

| フェーズ | 目的 | 代表的な手法・ツール |
|---|---|---|
| **Reconnaissance** | ターゲット情報の収集 | OSINT, Whois, DNS lookup, Google Dorks |
| **Scanning** | ポート・脆弱性の探索 | Nmap, Nessus, Nikto |
| **Gaining Access** | 脆弱性を突いた侵入 | Metasploit, SQLi, Buffer Overflow |
| **Maintaining Access** | 永続的なアクセスの確保 | Backdoor, Rootkit, RAT |
| **Clearing Tracks** | ログ削除・証拠の消去 | Log wiping, Timestomping |

---

## 偵察（Reconnaissance / Footprinting）

偵察は**Passive（非能動）**と**Active（能動）**に大別されます。

### Passive Reconnaissance（受動的偵察）

ターゲットに直接アクセスせず、公開情報を収集する。

| 用語 | 説明 |
|---|---|
| **OSINT** | Open Source Intelligence。公開情報源からの情報収集 |
| **Whois** | ドメイン登録情報（所有者・サーバー・連絡先）を取得 |
| **DNS Interrogation** | DNSレコードからIPアドレスやメールサーバーを特定 |
| **Google Dorks** | Google の高度な検索演算子で機密情報を探索 |
| **Shodan** | インターネット上のデバイスを検索できる特殊な検索エンジン |
| **EDGAR** | 米国企業の財務情報が公開されているデータベース |
| **Social Engineering** | 人的手法による情報収集（後述） |

### Active Reconnaissance（能動的偵察）

ターゲットに直接接触して情報を収集する。法的リスクを伴う。

| 用語 | 説明 |
|---|---|
| **Traceroute** | パケットの経路とホップ数を調査 |
| **Ping Sweep** | 複数のIPアドレスに対して生存確認を行う |
| **Social Engineering（対面）** | 電話・対面での情報引き出し |

---

## スキャン（Scanning & Enumeration）

### スキャンの種類

| スキャン種別 | 概要 |
|---|---|
| **Port Scanning** | 開いているTCP/UDPポートを特定 |
| **Vulnerability Scanning** | 既知の脆弱性を自動検出 |
| **Network Scanning** | 稼働中のホストとトポロジーを特定 |

### Nmap のスキャン技法

| オプション | スキャン種別 | 特徴 |
|---|---|---|
| `-sS` | SYN Scan（Stealth） | 3ウェイハンドシェイクを完了させない半開スキャン |
| `-sT` | TCP Connect Scan | 完全な接続を確立。ログに残りやすい |
| `-sU` | UDP Scan | UDPポートを探索。低速だが重要 |
| `-sF` | FIN Scan | FINフラグのみ送信。ファイアウォール回避に使用 |
| `-sX` | Xmas Scan | FIN+PSH+URGフラグ。Windowsには効かない |
| `-sN` | Null Scan | フラグなしのパケット。ステルス性が高い |
| `-sA` | ACK Scan | ファイアウォールのルールセットを調査するために使用 |
| `-O` | OS Detection | TTLやウィンドウサイズからOSを推定 |

### Enumeration（列挙）

スキャンで発見したサービスから、ユーザー名・共有リソース・設定情報をさらに詳細に収集するプロセス。

| プロトコル | ポート | 列挙できる情報 |
|---|---|---|
| **NetBIOS** | 137-139 | コンピュータ名、共有フォルダ |
| **SNMP** | 161 | デバイス情報、設定値（Community String が "public" の場合に注意） |
| **LDAP** | 389 | ユーザー・グループ・組織情報（Active Directory） |
| **SMB** | 445 | 共有フォルダ、ユーザー、グループ |
| **NTP** | 123 | ネットワーク内のホスト一覧 |
| **SMTP** | 25 | メールアカウントの存在確認（VRFY / EXPN コマンド） |
| **DNS Zone Transfer** | 53 | DNSゾーン内の全レコード |

---

## システムハッキング（System Hacking）

### パスワード攻撃

| 攻撃手法 | 説明 |
|---|---|
| **Dictionary Attack** | 辞書ファイルのパスワードリストを順番に試す |
| **Brute Force** | 全ての文字の組み合わせを試す。時間がかかる |
| **Hybrid Attack** | 辞書+数字・記号の付加など辞書とブルートフォースの組み合わせ |
| **Rule-based Attack** | 置換ルール（"a"→"@"）を適用しながら辞書を変形 |
| **Rainbow Table Attack** | 事前計算済みハッシュテーブルで高速にパスワードを逆引き |
| **Pass-the-Hash (PtH)** | パスワードのハッシュ値をそのまま認証に使用する（Windows NT Hash） |
| **Credential Stuffing** | 漏洩した認証情報を他のサービスに試す |
| **Password Spraying** | 少数のパスワードを大量のアカウントに試す（ロックアウト回避） |

### 権限昇格（Privilege Escalation）

| 種別 | 概要 |
|---|---|
| **Vertical Privilege Escalation** | 低権限ユーザーから管理者・rootへ昇格 |
| **Horizontal Privilege Escalation** | 同じ権限レベルで別ユーザーに成りすます |

**よく利用される技法：**
- SUID/SGID ビットの誤設定（Linux）
- DLL Hijacking / DLL Injection（Windows）
- Unquoted Service Path（Windows）
- Kernel Exploit

### 永続化（Maintaining Access）

| 手法 | 説明 |
|---|---|
| **Backdoor** | 正規の認証を迂回する秘密の入口 |
| **Rootkit** | OS の深部に潜み自身の存在を隠蔽するマルウェア |
| **Scheduled Task / Cron Job** | 定期的に実行されるタスクとしてマルウェアを登録 |
| **Registry Run Key** | Windows の自動起動レジストリキーへの登録 |

---

## マルウェア（Malware Threats）

### マルウェアの種類

| 種別 | 説明 |
|---|---|
| **Virus** | 自己複製し他のファイルに感染する。宿主ファイルが必要 |
| **Worm** | 宿主なしで自己複製・ネットワーク拡散する |
| **Trojan** | 正規ソフトウェアに偽装した悪意あるプログラム |
| **RAT（Remote Access Trojan）** | 遠隔から感染端末を制御するトロイの木馬 |
| **Ransomware** | ファイルを暗号化し身代金を要求する |
| **Spyware** | ユーザーの活動を秘密裏に監視・送信する |
| **Adware** | 広告を強制表示する。スパイウェアを兼ねる場合もある |
| **Keylogger** | キーストロークを記録しパスワードを窃取する |
| **Rootkit** | OS やファームウェアレベルで潜伏し検出を回避する |
| **Fileless Malware** | ディスクに書き込まず、メモリ上でのみ動作する |
| **Botnet** | C2（Command & Control）サーバーで遠隔制御される感染端末群 |

### マルウェア解析

| 分析手法 | 概要 |
|---|---|
| **Static Analysis** | 実行せずにバイナリ・コードを解析（strings, IDA Pro） |
| **Dynamic Analysis** | 実際に動かしてサンドボックスで挙動を観察（Cuckoo Sandbox） |
| **Hybrid Analysis** | 静的＋動的を組み合わせた総合解析 |

---

## ソーシャルエンジニアリング（Social Engineering）

技術的な脆弱性ではなく**人間の心理的弱点**を突く攻撃群。

### 主な手法

| 手法 | 説明 |
|---|---|
| **Phishing** | 偽メール・偽サイトで認証情報をだまし取る |
| **Spear Phishing** | 特定の個人・組織を狙った精巧なフィッシング |
| **Whaling** | 経営幹部（クジラ）を標的にしたスピアフィッシング |
| **Vishing** | 音声通話（Voice + Phishing）による詐欺 |
| **Smishing** | SMS を使ったフィッシング |
| **Pretexting** | 架空のシナリオ（口実）を作り情報を引き出す |
| **Baiting** | 感染USBを置いたり無料コンテンツを提示する |
| **Quid Pro Quo** | 利益提供と引き換えに情報を得る（偽ITサポートなど） |
| **Tailgating / Piggybacking** | セキュリティエリアへの不正な物理的侵入 |
| **Dumpster Diving** | ごみ箱から機密情報を取得する |

### ソーシャルエンジニアリングの心理原則（Cialdini の 6 原則）

1. **Reciprocity（返報性）** - お返しをしなければという心理
2. **Commitment（一貫性）** - 一度決めたことを変えたくない心理
3. **Social Proof（社会的証明）** - 皆がやっているから正しい
4. **Authority（権威）** - 権威ある人物の指示に従う
5. **Liking（好意）** - 好きな人の頼みを断りにくい
6. **Scarcity（希少性）** - 限定・今だけという焦りを生む

---

## ネットワーク攻撃

### スニッフィングとARP

| 用語 | 説明 |
|---|---|
| **Passive Sniffing** | HUB 環境でブロードキャストトラフィックを傍受 |
| **Active Sniffing** | スイッチ環境でARPポイズニングなどを使い強制的に傍受 |
| **ARP Spoofing / Poisoning** | 偽ARPリプライで通信を攻撃者経由に誘導（MitM） |
| **MAC Flooding** | スイッチのCAMテーブルを溢れさせ、HUBモードにさせる |
| **DHCP Starvation** | 偽のDHCPリクエストを大量送信してアドレスプールを枯渇させる |
| **Rogue DHCP Server** | 偽DHCPサーバーで悪意あるDNS・デフォルトGWを配布する |

### DoS / DDoS 攻撃

| 攻撃 | 概要 |
|---|---|
| **SYN Flood** | TCPハンドシェイクの SYN パケットを大量送信し接続テーブルを枯渇させる |
| **Smurf Attack** | 偽装したブロードキャストPingで被害者へ大量応答を集中させる |
| **Fraggle Attack** | SmufrのUDP版（ポート7/19を利用） |
| **Slowloris** | HTTPヘッダを完成させずに接続を維持し続けてサーバーを枯渇させる |
| **RUDY** | POST データをゆっくり送ることでサーバーをブロックする |
| **Ping of Death** | 規格外の大きさのICMPパケットでシステムをクラッシュさせる（古典的） |
| **Teardrop** | 断片化されたパケットの再組立エラーを引き起こす |
| **Amplification Attack** | DNS・NTPなどを使いリクエストの何倍もの応答を被害者へ集中させる |
| **Botnet / DDoS** | ボットネットを使った分散型サービス拒否攻撃 |

### MitM（中間者攻撃）

| 手法 | 説明 |
|---|---|
| **SSL Stripping** | HTTPS→HTTP にダウングレードしトラフィックを傍受 |
| **HTTPS Spoofing** | 類似ドメインのHTTPS証明書を使いフィッシング |
| **LLMNR / NBT-NS Poisoning** | ローカルの名前解決プロトコルを乗っ取り認証情報を取得（Responder） |
| **BGP Hijacking** | ルーティングテーブルを操作してトラフィックをリダイレクト |

---

## Webアプリケーション攻撃

### OWASP Top 10（2021）

| 順位 | 脆弱性 | 概要 |
|---|---|---|
| A01 | **Broken Access Control** | 認可の不備。機能・データへの不正アクセス |
| A02 | **Cryptographic Failures** | 機密データの暗号化不足・平文送信 |
| A03 | **Injection** | SQLi, XSS, コマンドインジェクション |
| A04 | **Insecure Design** | 設計段階のセキュリティ考慮不足 |
| A05 | **Security Misconfiguration** | デフォルト設定・不要機能の放置 |
| A06 | **Vulnerable Components** | 脆弱なライブラリ・フレームワーク |
| A07 | **Identification & Auth Failures** | 認証・セッション管理の欠陥 |
| A08 | **Software Integrity Failures** | 署名なし更新・CI/CDの信頼の欠如 |
| A09 | **Logging & Monitoring Failures** | ログ不足による侵害検出の遅れ |
| A10 | **SSRF** | サーバーを踏み台にした内部リクエスト |

### 主要な攻撃手法

#### SQL インジェクション

```sql
-- 認証バイパス（Tautology）
' OR '1'='1

-- UNIONを使ったデータ抽出
' UNION SELECT username, password FROM users--

-- ブラインドSQLi（真偽判定）
' AND 1=1--   -- True（正常応答）
' AND 1=2--   -- False（エラー or 応答変化）

-- 時間ベースのブラインドSQLi
' AND SLEEP(5)--
```

| SQLi の種別 | 説明 |
|---|---|
| **In-band SQLi** | 同じチャネルで結果を受け取る（Error-based, Union-based） |
| **Blind SQLi** | 結果が見えない（Boolean-based, Time-based） |
| **Out-of-band SQLi** | DNSや外部HTTPリクエスト経由で結果を受け取る |

#### XSS（Cross-Site Scripting）

| 種別 | 説明 |
|---|---|
| **Reflected XSS** | URLパラメータがそのままレスポンスに反映される |
| **Stored XSS** | データベースに保存され全ユーザーに配信される（永続型） |
| **DOM-based XSS** | サーバーを介さず、クライアント側のDOM操作で発生 |

#### CSRF（Cross-Site Request Forgery）

認証済みユーザーに、攻撃者が用意したページから意図しないリクエストを送信させる攻撃。CSRFトークンや SameSite Cookie で対策。

#### IDOR（Insecure Direct Object Reference）

URL の `id=123` を `id=124` に変更するだけで他人のデータが見える脆弱性。アクセス制御の欠如が原因。

#### SSRF（Server-Side Request Forgery）

```
# 内部メタデータへのアクセス（クラウド環境）
http://169.254.169.254/latest/meta-data/

# 内部サービスへのアクセス
http://localhost:8080/admin
```

---

## セッションハイジャック

| 用語 | 説明 |
|---|---|
| **Session Hijacking** | 正規ユーザーのセッションIDを盗用してなりすます |
| **Session Fixation** | 攻撃者が指定したセッションIDをユーザーに使わせる |
| **Cookie Theft** | XSSやスニッフィングでCookieを盗む |
| **TCP Session Hijacking** | シーケンス番号を予測してTCPセッションを乗っ取る |
| **Man-in-the-Browser** | ブラウザ拡張機能やトロイの木馬がブラウザ内部に潜む |

---

## ファイアウォール・IDS/IPS 回避

### IDS / IPS の種類

| 種別 | 説明 |
|---|---|
| **NIDS** | ネットワーク型。トラフィック全体を監視 |
| **HIDS** | ホスト型。ログ・プロセス・ファイルを監視 |
| **Signature-based** | 既知パターンと照合。ゼロデイには無力 |
| **Anomaly-based** | 正常なベースラインから逸脱を検出 |
| **Stateful Protocol Analysis** | プロトコルの正常な状態遷移と照合 |

### IDS 回避技法

| 技法 | 説明 |
|---|---|
| **Fragmentation** | パケットを細分化してシグネチャを分断する |
| **TTL Manipulation** | TTLを操作してIDSには届くがターゲットには届かないパケットを使う |
| **Unicode Encoding** | URLエンコードなどで攻撃文字列を隠蔽する |
| **Session Splicing** | TCPストリームを分断して再組立でシグネチャを回避 |
| **Obfuscation / Polymorphism** | コードを難読化・変形させてシグネチャを変える |
| **DoS against IDS** | IDSそのものを過負荷にさせる |

---

## 暗号化（Cryptography）

### 対称鍵 vs 非対称鍵

| 方式 | 代表アルゴリズム | 特徴 |
|---|---|---|
| **対称鍵（Symmetric）** | AES, DES, 3DES, Blowfish | 同一キーで暗号化・復号。高速だがキー配送問題がある |
| **非対称鍵（Asymmetric）** | RSA, ECC, Diffie-Hellman | 公開鍵・秘密鍵ペア。遅いが鍵配送が安全 |

### 主要なアルゴリズム

| アルゴリズム | 種別 | キーサイズ | 用途 |
|---|---|---|---|
| **AES** | 対称 | 128/192/256 bit | 現代の標準。TLS、ファイル暗号化 |
| **DES** | 対称 | 56 bit | 廃止済み。ブルートフォース可能 |
| **3DES** | 対称 | 168 bit（実効112） | DESを3回適用。2023年以降非推奨 |
| **RSA** | 非対称 | 2048/4096 bit | 鍵交換、デジタル署名 |
| **ECC** | 非対称 | 256 bit相当 | RSAより短いキーで同等の安全性 |
| **MD5** | ハッシュ | 128 bit出力 | 衝突耐性が破られ廃止済み |
| **SHA-1** | ハッシュ | 160 bit出力 | 衝突耐性が破られ非推奨 |
| **SHA-256** | ハッシュ | 256 bit出力 | 現在の標準 |
| **SHA-3** | ハッシュ | 224〜512 bit | SHA-2の後継候補 |

### PKI（Public Key Infrastructure）

| 用語 | 説明 |
|---|---|
| **CA（Certificate Authority）** | デジタル証明書を発行・管理する認証局 |
| **Root CA** | PKI の信頼の起点となる最上位CA |
| **Intermediate CA** | Root CA の下位に位置する中間CA |
| **CRL（Certificate Revocation List）** | 失効した証明書のリスト |
| **OCSP** | リアルタイムで証明書の有効性を確認するプロトコル |
| **Digital Signature** | 送信者の秘密鍵で署名し完全性と否認防止を確保 |

### 主要プロトコルのポート番号

| プロトコル | ポート | 説明 |
|---|---|---|
| HTTP | 80 | 平文Web通信 |
| **HTTPS** | 443 | TLSで暗号化されたWeb通信 |
| FTP | 21（制御）/ 20（データ） | ファイル転送 |
| **SFTP** | 22 | SSH上のセキュアなファイル転送 |
| SSH | 22 | セキュアリモートシェル |
| Telnet | 23 | 平文リモートシェル（非推奨） |
| SMTP | 25 | メール送信 |
| DNS | 53 | ドメイン名解決 |
| DHCP | 67/68 | IPアドレスの動的割り当て |
| SNMP | 161/162 | ネットワーク機器の管理 |
| LDAP | 389 | ディレクトリサービス |
| RDP | 3389 | Windowsリモートデスクトップ |
| **LDAPS** | 636 | TLS上のLDAP |

---

## クラウドセキュリティ

### クラウドのサービスモデルと責任分界

```
SaaS: データ・アクセス制御は顧客責任
IaaS: OS以上の全レイヤーが顧客責任
PaaS: アプリケーションとデータが顧客責任
```

### 主なクラウド攻撃

| 攻撃 | 説明 |
|---|---|
| **IAM Misconfiguration** | 過剰な権限付与によるリソースへの不正アクセス |
| **S3 Bucket Misconfiguration** | 公開設定のまま放置されたストレージからのデータ漏洩 |
| **Metadata API Abuse** | `169.254.169.254` からIAMクレデンシャルを取得 |
| **Container Escape** | コンテナの隔離を突破しホストOSへ侵入 |
| **Cryptojacking** | クラウドリソースで不正に暗号通貨をマイニング |

---

## モバイル・IoT セキュリティ

### Android のセキュリティ

| 用語 | 説明 |
|---|---|
| **Rooting** | スーパーユーザー権限を取得すること |
| **APK Repackaging** | 正規アプリを改変・マルウェアを埋め込んで再配布 |
| **Sideloading** | 公式ストア外からAPKをインストール |
| **ADB（Android Debug Bridge）** | 開発・デバッグ用の強力なCLIツール。悪用される場合も |

### iOS のセキュリティ

| 用語 | 説明 |
|---|---|
| **Jailbreaking** | Apple の署名制限を回避してルートアクセスを取得 |
| **Entitlement Abuse** | アプリに付与された特権の悪用 |

### IoT 攻撃

| 攻撃 | 説明 |
|---|---|
| **Default Credentials** | 変更されていないデフォルトのID/パスワードを利用 |
| **Firmware Analysis** | ファームウェアを抽出・逆アセンブルして脆弱性を探す |
| **MQTT Eavesdropping** | IoTメッセージングプロトコルへの盗聴（認証なしの場合） |

---

## ペネトレーションテスト（Pen Testing）

### ペンテストの分類

| 種別 | 説明 |
|---|---|
| **Black Box** | ターゲットに関する事前情報なし（実際の攻撃者の視点） |
| **White Box** | ソースコード・ネットワーク図など全情報を提供（完全情報） |
| **Gray Box** | 部分的な情報のみ提供（中間） |

### ペンテストのフロー

```
1. 契約・スコープ定義（Rules of Engagement）
2. 偵察（Reconnaissance）
3. スキャン・列挙（Scanning & Enumeration）
4. 脆弱性分析（Vulnerability Analysis）
5. 悪用・侵入（Exploitation）
6. 事後活動（Post-Exploitation）
7. 痕跡消去（Cleanup）
8. 報告書作成（Reporting）
```

### 主要ツール一覧

| ツール | 用途 |
|---|---|
| **Nmap** | ポートスキャン・OS検出 |
| **Metasploit Framework** | 脆弱性の実証・エクスプロイト |
| **Burp Suite** | Webアプリケーションのプロキシ・テスト |
| **Wireshark** | パケットキャプチャ・解析 |
| **Aircrack-ng** | Wi-Fi の暗号解読 |
| **Hashcat / John the Ripper** | パスワードハッシュのクラック |
| **Hydra / Medusa** | ブルートフォース・クレデンシャルスタッフィング |
| **Nikto** | Webサーバーの脆弱性スキャン |
| **SQLMap** | SQLインジェクション自動検出・悪用 |
| **Maltego** | OSINT・リレーション可視化 |
| **theHarvester** | メールアドレス・サブドメイン収集 |
| **Mimikatz** | Windowsのメモリからクレデンシャルを抽出 |
| **BeEF** | ブラウザを利用したエクスプロイトフレームワーク |

---

## インシデント対応とフォレンジック

### インシデント対応のフロー（PICERL）

```
Preparation（準備）
    ↓
Identification（識別）
    ↓
Containment（封じ込め）
    ↓
Eradication（根絶）
    ↓
Recovery（復旧）
    ↓
Lessons Learned（事後学習）
```

### デジタルフォレンジック

| 用語 | 説明 |
|---|---|
| **Chain of Custody** | 証拠の取り扱い記録。法廷での証拠能力を保証する |
| **Forensic Image** | ディスクのビット単位のコピー（dd, FTK Imager） |
| **Volatile Data** | RAM上のデータ。電源OFF前に最優先で収集 |
| **Hash Verification** | 証拠の改ざんを防ぐためMD5/SHA256でハッシュを記録 |
| **Write Blocker** | フォレンジック時に証拠媒体への書き込みを防ぐ装置 |

---

## 重要な数値と定義の早見表

| 事項 | 値 |
|---|---|
| **RC4 キーサイズ** | 40〜2048 bit |
| **AES ブロックサイズ** | 128 bit（固定） |
| **RSA 推奨キーサイズ** | 2048 bit 以上 |
| **TCP 3ウェイハンドシェイク** | SYN → SYN-ACK → ACK |
| **OSI モデル** | 7層（物理・データリンク・ネットワーク・トランスポート・セッション・プレゼンテーション・アプリケーション） |
| **TCP ポート範囲** | 0〜65535（Well-Known: 0〜1023） |

---

## 参考リソース

### 書籍

| 書籍 | 著者 | 特徴 |
|---|---|---|
| **The Web Application Hacker's Handbook** | Stuttard / Pinto | Web 攻撃の仕組みを深く掘り下げる。実務にも直結 |
| **Hacking: The Art of Exploitation** | Jon Erickson | バッファオーバーフロー・シェルコードを C で学ぶ。上級者向け |
| **ネットワークはなぜつながるのか** | 戸根勤 | TCP/IP の基礎を日本語で丁寧に解説。ネットワーク基礎固めに最適 |

---

### 無料の学習プラットフォーム

#### ハンズオン演習（脆弱な環境・CTF）

| プラットフォーム | 特徴 |
|---|---|
| **TryHackMe** | ブラウザだけで始められる。初心者から中級者向けの体系的なラーニングパスが充実 |
| **Hack The Box** | 実践的な CTF マシン群。中〜上級者向け |
| **PentesterLab** | Web 脆弱性に特化。SQLi・XSS・SSRF などを段階的に学べる |
| **PortSwigger Web Security Academy** | Burp Suite 開発元による無料の Web セキュリティ教材。OWASP 全項目をカバー |
| **DVWA（Damn Vulnerable Web App）** | ローカルに構築できる脆弱なWebアプリ。SQLi/XSS を安全に練習できる |
| **WebGoat** | OWASP が提供する意図的に脆弱なWebアプリ学習環境 |
| **VulnHub** | 脆弱な仮想マシンのダウンロードサイト。ローカルで Pen Test を練習 |

#### 動画・講座

| プラットフォーム | 特徴 |
|---|---|
| **Udemy（Heath Adams / TCM Security）** | 実践的なエシカルハッキングコース。セール時に安価で購入可能 |
| **INE（eLearnSecurity）** | ペネトレーションテスト・レッドチームに対応した体系的なカリキュラム |
| **YouTube: NetworkChuck** | ネットワークとセキュリティを楽しく解説。Nmap・Wireshark の入門動画が豊富 |
| **YouTube: John Hammond** | CTF の解法・マルウェア解析を実演。中上級者向け |
| **Cybrary** | セキュリティ資格対策の無料・有料コースを提供 |

---

### ツールの練習環境を素早く構築する

```bash
# Kali Linux を Docker で即起動（ローカルにインストール不要）
docker pull kalilinux/kali-rolling
docker run -it kalilinux/kali-rolling /bin/bash

# DVWA をローカルで立ち上げる
docker pull vulnerables/web-dvwa
docker run -d -p 80:80 vulnerables/web-dvwa
# → http://localhost にアクセスし admin/password でログイン

# WebGoat を立ち上げる
docker pull webgoat/goat-and-wolf
docker run -d -p 8080:8080 webgoat/goat-and-wolf
# → http://localhost:8080/WebGoat
```

---

### 最新の脅威情報を継続的にフォローする

| リソース | 種別 | 説明 |
|---|---|---|
| **CVE（mitre.org）** | 脆弱性DB | 公式の脆弱性識別番号と詳細 |
| **NVD（nvd.nist.gov）** | 脆弱性DB | CVE に CVSS スコアを付加した NIST のデータベース |
| **Exploit-DB** | エクスプロイトDB | 公開 PoC コードの検索。`searchsploit` でローカル検索可能 |
| **OWASP.org** | Web セキュリティ | Top 10・チートシート・テストガイドを無料公開 |
| **SANS Internet Storm Center** | 脅威情報 | 日次の脅威レポートと技術解説 |
| **Krebs on Security** | ニュース | Brian Krebs による深掘りセキュリティレポート |
| **The Hacker News** | ニュース | 最新の脆弱性・インシデント情報を速報 |

---

## まとめ：学習ロードマップ

```
Step 1: フェーズとスキャン技法を固める
         → 5フェーズの流れと Nmap のオプションを理解する
         → TryHackMe の「Pre-Security」パスで基礎を補完

Step 2: ネットワーク攻撃を体系化
         → ARP/DoS/MitM の仕組みを図で理解する
         → Wireshark でパケットキャプチャを実際に観察する

Step 3: Web 攻撃を実践で確認
         → SQLi, XSS, CSRF を DVWA / WebGoat で体験する
         → PortSwigger Web Security Academy を平行して進める

Step 4: マルウェア・暗号化を整理
         → 種別の違い、アルゴリズムの特性を表で整理する
         → TryHackMe の「SOC Level 1」でマルウェア解析に触れる

Step 5: フォレンジックとインシデント対応を補完
         → 証拠保全の手順、インシデント対応フローを押さえる
         → ペネトレーションテストの契約・スコープ定義の考え方を理解する
```

用語の暗記だけでなく、**攻撃の流れと防御策をセットで理解する**ことがレッドチーム学習の本質です。得た知識は倫理的・合法的な範囲でのみ活用してください。

:::message alert
本記事の内容を**無許可の環境に適用することは違法**です。学習・研究・CTF・合法的なペネトレーションテスト以外への転用は絶対にしないでください。
:::

---

*この記事は継続的に更新する予定です。誤りや補足があればコメントで教えてください。*
