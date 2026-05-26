---
layout: post
title: "レッドチームセキュリティ学習：ハッキングツールを5フェーズで整理"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["security", "redteam", "pentest", "ethical-hacking", "tools"]
date: "2026-05-26 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-17-ceh-tools-guide)でも公開しています。


## はじめに

セキュリティの**レッドチーム（攻撃側）**の視点を学ぶうえで、攻撃の各フェーズでどのようなツールが使われるかを体系的に理解することは非常に重要です。攻撃者の思考・手法を知ることで、より実効性の高い防御設計や脅威モデリングができるようになります。

この記事は、ペネトレーションテストで広く使われる**攻撃5フェーズ**のフレームワークに沿って、代表的なセキュリティツールを整理した学習ノートです。**レッドチーム／ペネトレーションテストの概念を体系的に把握したい方**を対象としています。

:::message alert
**重要：悪用禁止**

本記事は**セキュリティ学習・研究目的のまとめ**です。記載のツールや技術を**許可なく他者のシステムに使用することは不正アクセス禁止法等に違反する犯罪行為**です。

- 使用は必ず**自分が管理・所有する環境**、または**明示的な許可を得たペネトレーションテスト環境・CTF**に限定してください
- **絶対に悪用しないでください**
:::


---

## 攻撃5フェーズ とツールの対応

```
Phase 1: Reconnaissance（偵察）        ← 情報収集
Phase 2: Scanning（スキャン）           ← ポート・脆弱性調査
Phase 3: Gaining Access（侵入）         ← エクスプロイト
Phase 4: Maintaining Access（維持）     ← バックドア・持続化
Phase 5: Covering Tracks（痕跡消去）    ← ログ削除・隠蔽
```

---

## Phase 1: Reconnaissance（偵察）

### パッシブ偵察（対象に触れない）

| ツール | 用途 | 特徴 |
|---|---|---|
| **Maltego** | OSINT・関係マッピング | グラフUIで人・ドメイン・IP の関係を可視化 |
| **Shodan** | インターネット公開機器の検索 | バナー情報・サービス・脆弱バージョンを検索 |
| **theHarvester** | メール・ドメイン・サブドメイン収集 | Google/Bing/LinkedIn など複数ソースを自動クロール |
| **Recon-ng** | OSINT フレームワーク | Metasploit ライクなモジュール構成 |
| **FOCA** | メタデータ抽出 | PDF/Word ファイルからメタデータ（作成者・パスなど）を取得 |
| **Censys** | Shodan の代替・証明書情報も取得 | TLS 証明書経由でのサブドメイン列挙に有効 |
| **SpiderFoot** | 自動 OSINT | メール・IP・ドメイン・SNS を横断収集 |

### アクティブ偵察（対象に接触する）

| ツール | 用途 | 特徴 |
|---|---|---|
| **Nslookup / Dig** | DNS 照会 | A/MX/NS/AAAA/PTR レコードの確認 |
| **Whois** | ドメイン登録情報取得 | 登録者・ネームサーバー・有効期限を確認 |
| **Traceroute / Tracert** | 経路追跡 | パケットの経路と RTT をホップごとに確認 |
| **Netcraft** | Web サーバー情報調査 | OS・Webサーバーの種類とバージョンを取得 |

---

## Phase 2: Scanning（スキャン）

### ポートスキャン

| ツール | 用途 | 特徴 |
|---|---|---|
| **Nmap** | ポート・サービス・OS スキャン | ペネトレーションテストの定番ツール。TCP/UDP/SYN/Xmas など多彩なスキャン方式 |
| **Zenmap** | Nmap の GUI フロントエンド | Nmap コマンドを視覚的に操作・可視化 |
| **Angry IP Scanner** | 軽量ネットワークスキャナ | IP レンジを高速にスキャン |
| **Masscan** | 超高速ポートスキャナ | インターネット全体を数分でスキャン可能 |
| **Unicornscan** | 非同期ポートスキャナ | UDP スキャンにも強い |

### Nmap 主要フラグ早見表

| フラグ | スキャン種別 | 説明 |
|---|---|---|
| `-sS` | SYN Scan（Stealth） | 接続を完了させずにポートを検出（最もよく使われる） |
| `-sT` | TCP Connect Scan | 完全接続（ログに残りやすい） |
| `-sU` | UDP Scan | UDP ポートのスキャン |
| `-sX` | Xmas Scan | FIN+PSH+URG フラグを立てる |
| `-sN` | Null Scan | フラグをすべてオフにする |
| `-sF` | FIN Scan | FIN フラグのみ |
| `-O` | OS Detection | TTL などからOSを推定 |
| `-sV` | Service Version | サービスのバージョンを取得 |
| `-A` | Aggressive | OS+Version+Script+Traceroute |
| `-p-` | 全ポートスキャン | 65535 ポート全てを対象にする |

### 脆弱性スキャン

| ツール | 用途 | 特徴 |
|---|---|---|
| **Nessus** | 脆弱性スキャナ | 業界標準。CVSS スコアで重大度を分類 |
| **OpenVAS** | OSS 脆弱性スキャナ | Nessus の代替オープンソース版 |
| **Nexpose** | Rapid7 製脆弱性管理 | Metasploit と連携可能 |
| **Qualys** | クラウド型脆弱性管理 | エージェントレス・SaaS で提供 |
| **Retina** | eEye 製スキャナ | パッチ管理機能付き |

### 列挙（Enumeration）

| ツール | 用途 | 特徴 |
|---|---|---|
| **Enum4linux** | SMB/NetBIOS 列挙 | ユーザー・共有・グループ情報を取得 |
| **NBTscan** | NetBIOS スキャン | NetBIOS 名前テーブルを収集 |
| **SNMPwalk** | SNMP 列挙 | MIB ツリーを再帰的に取得 |
| **ldapsearch** | LDAP 列挙 | Active Directory のユーザー・OU 情報を取得 |
| **Gobuster / Dirbuster** | Web ディレクトリ列挙 | ワードリストで隠しパスを総当たり |
| **wfuzz** | Web ファジング | パラメータ・パス・ヘッダーのファジング |

---

## Phase 3: Gaining Access（侵入）

### エクスプロイトフレームワーク

| ツール | 用途 | 特徴 |
|---|---|---|
| **Metasploit Framework** | エクスプロイト・ペイロード実行 | レッドチームの代表的フレームワーク。2000超のモジュールを内蔵 |
| **BeEF (Browser Exploitation Framework)** | ブラウザ経由の攻撃 | XSS でフックしブラウザを制御 |
| **Exploit-DB / searchsploit** | エクスプロイトデータベース | CVE に対応するエクスプロイトコードを検索 |

#### Metasploit 主要コマンド

```bash
msfconsole             # 起動
search <keyword>       # モジュール検索
use <module>           # モジュール選択
show options           # オプション一覧
set RHOSTS <target>    # ターゲット設定
set PAYLOAD <payload>  # ペイロード選択
run / exploit          # 実行
sessions -l            # セッション一覧
sessions -i <id>       # セッションに接続
```

### パスワードクラック

| ツール | 手法 | 特徴 |
|---|---|---|
| **Hashcat** | オフラインクラック（GPU） | 世界最速。MD5/NTLM/bcrypt など多数のハッシュ対応 |
| **John the Ripper** | オフラインクラック | ルールベース攻撃が強力。Unix Shadow ファイルに強い |
| **Hydra** | オンライン総当たり | SSH/FTP/HTTP/RDP/SMB など50以上のプロトコル対応 |
| **Medusa** | オンライン総当たり | Hydra の代替。並列処理が高速 |
| **Aircrack-ng** | Wi-Fi ハンドシェイククラック | WPA/WPA2 の 4-way ハンドシェイクを解析 |
| **CeWL** | カスタムワードリスト生成 | ターゲットサイトをクロールして単語帳を生成 |
| **Crunch** | ワードリスト生成 | 文字種・長さを指定してリスト生成 |

#### パスワード攻撃の種類

| 攻撃種別 | 説明 |
|---|---|
| **Dictionary Attack** | 既知の単語リストを使用 |
| **Brute Force** | 全組み合わせを試行 |
| **Rule-based Attack** | 変換ルール（leet 置換など）を適用 |
| **Rainbow Table** | 事前計算済みハッシュテーブルと比較 |
| **Pass-the-Hash (PtH)** | ハッシュそのものを認証に使用（Windowsネットワーク） |

### ソーシャルエンジニアリング

| ツール | 用途 | 特徴 |
|---|---|---|
| **SET (Social-Engineer Toolkit)** | フィッシング・クローンサイト | 最も使われる SE フレームワーク |
| **Gophish** | フィッシングキャンペーン管理 | WebUI でメール送信・追跡が可能 |
| **King Phisher** | フィッシングメール送信 | SMTP 設定が柔軟 |

### Webアプリケーション攻撃

| ツール | 用途 | 特徴 |
|---|---|---|
| **Burp Suite** | Web プロキシ・脆弱性診断 | Webアプリ診断の業界標準。インターセプト・リピーター・スキャナ機能 |
| **OWASP ZAP** | OSS Web スキャナ | Burp Suite の OSS 代替 |
| **SQLmap** | SQL インジェクション自動化 | DB の種類を自動判別し盲目的なSQLiも検出 |
| **Nikto** | Web サーバースキャナ | 既知の脆弱設定・ファイルを自動検出 |
| **w3af** | Web アプリケーションフレームワーク | 200以上のプラグインで診断 |
| **XSStrike** | XSS 検出・エクスプロイト | コンテキストを分析して高精度のペイロード生成 |

### ワイヤレス攻撃

| ツール | 用途 | 特徴 |
|---|---|---|
| **Aircrack-ng スイート** | Wi-Fi 攻撃の総合ツール | Monitor mode → Capture → Crack の一連を担う |
| **Airodump-ng** | パケットキャプチャ | BSSID・ESSID・CH・クライアントを一覧化 |
| **Aireplay-ng** | パケットインジェクション | Deauth 攻撃でハンドシェイクを強制取得 |
| **Kismet** | 無線LAN 探索・モニタリング | 隠しSSID の検出も可能 |
| **Wifite** | Wi-Fi 自動攻撃 | WEP/WPA/WPS を自動で試行 |
| **Fern Wifi Cracker** | GUI Wi-Fi クラッカー | Aircrack-ng をGUIで操作 |

#### WPA2 クラックの概念フロー（各ステップとツールの対応）

レッドチーム学習では「どのツールがどのステップを担うか」を理解することが重要です。

| ステップ | 役割 | 使用ツール |
|---|---|---|
| ① モニターモード有効化 | 通常の通信を傍受可能な状態にする | `airmon-ng` |
| ② AP・クライアント探索 | 対象ネットワークの BSSID/CH を特定 | `airodump-ng` |
| ③ ハンドシェイクキャプチャ | 4-way ハンドシェイクをファイルに保存 | `airodump-ng` |
| ④ 再接続誘発（オプション） | クライアントを切断して再接続させる | `aireplay-ng` |
| ⑤ オフラインクラック | キャプチャしたハンドシェイクにワードリストを照合 | `aircrack-ng` / `hashcat` |

**防御側の視点：** WPA3 への移行、強力なパスフレーズ（20文字以上）の使用、Deauth フレームを検知する WIDS（無線侵入検知システム）の導入が対策として有効です。

---

## Phase 4: Maintaining Access（持続化）

| ツール | 用途 | 特徴 |
|---|---|---|
| **Meterpreter** | 高機能リモートシェル | Metasploit のペイロード。メモリ常駐・ファイルレス |
| **Netcat (nc)** | リバースシェル・バインドシェル | ネットワークの「スイスアーミーナイフ」 |
| **Cobalt Strike** | C2（Command & Control）フレームワーク | 商用APTシミュレーションツール |
| **Empire** | PowerShell C2 フレームワーク | エージェントレスでの Windows 権限昇格 |
| **Pupy** | クロスプラットフォーム RAT | Python ベース・メモリ内実行 |
| **Weevely** | PHP ウェブシェル | ターゲットWebサーバーへのバックドア |

### 権限昇格（Privilege Escalation）

| ツール | 用途 | 特徴 |
|---|---|---|
| **LinPEAS / WinPEAS** | 権限昇格のチェック自動化 | SUID/Cron/設定ミスなど数百の確認点を自動スキャン |
| **PowerSploit** | PowerShell 攻撃フレームワーク | `Invoke-Mimikatz` など強力なモジュール群 |
| **Mimikatz** | Windows 認証情報抽出 | LSASS から平文パスワード・NTLM ハッシュを取得 |
| **BeRoot** | Linux/Windows 権限昇格チェック | sudo ミス・PATH ハイジャックなどを検出 |

---

## Phase 5: Covering Tracks（痕跡消去）と防御

このフェーズでは、攻撃者が「何を狙って消そうとするか」を理解し、**それを守る防御策**をセットで学ぶことがレッドチーム学習の本質です。

### 攻撃者が標的にするログ・痕跡と防御策

| 攻撃者の標的 | 使われる手法 | 防御策・SOC での検知ポイント |
|---|---|---|
| **Windows イベントログ** | ログのクリア・監査ポリシー無効化 | Event ID 1102（ログクリア）・Event ID 4719（監査ポリシー変更）をアラート設定 |
| **ファイルのタイムスタンプ** | タイムスタンプ改ざん（Timestomp） | ファイル整合性監視（FIM）ツールでハッシュ変化を検知 |
| **syslog / auth.log** | 特定 IP のエントリ削除 | ログをリモートの集中 SIEM にリアルタイム転送して改ざんを無効化 |
| **ファイル痕跡** | 上書き削除（shred など） | 削除イベントの監視・バックアップによる復元 |
| **通信の隠蔽** | ステガノグラフィ（画像内への埋め込み） | DLP・コンテンツ検査による異常なバイナリ構造の検出 |

### 防御の基本原則

| 原則 | 内容 |
|---|---|
| **ログの集中管理** | ローカルログは改ざんされうる。SIEM にリアルタイム転送し不変性を確保する |
| **監査ポリシーの保護** | 監査ポリシー変更自体をログ・アラート対象にする |
| **FIM（ファイル整合性監視）** | 重要ファイルのハッシュを定期比較し改ざんを検知する |
| **最小権限の原則** | ログ削除・監査設定変更は管理者権限を要する。権限の最小化で被害範囲を限定する |

---

## ネットワーク盗聴・MITM

| ツール | 用途 | 特徴 |
|---|---|---|
| **Wireshark** | パケット解析 | GUI でキャプチャ・フィルタリング・解析 |
| **tcpdump** | CLI パケットキャプチャ | スクリプトや自動化に向く |
| **Ettercap** | ARP スプーフィング・MITM | LAN 内の通信を傍受・改ざん |
| **Bettercap** | Ettercap の後継 MITM ツール | HTTPS も含めた傍受。モジュラー設計 |
| **Responder** | LLMNR/NBT-NS 毒化 | Windows 環境のハッシュキャプチャに有効 |
| **MITMf** | MITM フレームワーク | SSLstrip・DNS スプーフィングなど統合 |
| **Arpspoof** | ARP テーブル汚染 | dsniff スイートの一部 |

---

## IDS/ファイアウォール回避

| ツール / 手法 | 用途 | 特徴 |
|---|---|---|
| **Nmap フラグ操作** | デコイスキャン・断片化 | `-D RND:10`（デコイ）`-f`（断片化）`-T0`（遅延） |
| **Proxychains** | プロキシ経由で接続 | Tor や SOCKS プロキシを連鎖させる |
| **Tor** | 匿名通信 | オニオンルーティングで送信元を隠蔽 |
| **VPN / SSH トンネル** | 暗号化トンネル | IDS がパケット内容を読めなくする |
| **Veil Framework** | ペイロード難読化 | AV 検知を回避する Meterpreter ペイロード生成 |
| **Shellter** | PE ファイル注入 | 既存の Windows EXE にシェルコードを埋め込む |

---

## フォレンジック・ステガノグラフィ

| ツール | 用途 | 特徴 |
|---|---|---|
| **Autopsy** | デジタルフォレンジック | The Sleuth Kit のGUIフロントエンド |
| **FTK (Forensic Toolkit)** | 商用フォレンジックスイート | AccessData 製。証拠収集・解析・レポート |
| **Volatility** | メモリフォレンジック | メモリダンプからプロセス・ネットワーク・パスワードを解析 |
| **dd / dcfldd** | ビット単位ディスクコピー | フォレンジックイメージ取得 |
| **Steghide** | ステガノグラフィ埋め込み・抽出 | JPEG/BMP/WAV に情報を隠蔽 |
| **Stegdetect** | ステガノグラフィ検出 | 統計解析でステガノグラフィの存在を推定 |
| **ExifTool** | メタデータ確認・編集 | 画像・ドキュメントのメタデータ全般に対応 |

---

## DoS/DDoS ツール

:::message alert
DoS/DDoS 攻撃ツールは**許可を得た環境以外での使用は違法**です。セキュリティ学習では概念・防御の理解を目的として取り上げています。
:::

| ツール | 攻撃種別 | 特徴 |
|---|---|---|
| **LOIC (Low Orbit Ion Cannon)** | UDP/TCP/HTTP フラッド | 匿名化なし・シンプルな負荷生成 |
| **HOIC (High Orbit Ion Cannon)** | HTTP フラッド | スクリプト可能・分散攻撃向け |
| **hping3** | カスタムパケット送信 | SYN フラッド・スマーフ攻撃の検証に使用 |
| **Slowloris** | 低速HTTP攻撃 | 少ないリソースでWebサーバーを枯渇させる |
| **R.U.D.Y.** | HTTP POST 低速攻撃 | ボディを極めてゆっくり送信しスレッドを占有 |

---

## ツールを体系的に理解するためのポイント

### 学習時に意識したい視点

1. **用途を把握する** — 「Nessus は何のためのツールか？脆弱性スキャナとして何を検出するか？」
2. **フェーズとの対応を掴む** — 「Covering Tracks フェーズで攻撃者が狙うものは何か？防御側はどう検知するか？」
3. **コマンドと動作原理を紐づける** — 「Nmap の `-sS` フラグは SYN スキャン。なぜステルスと呼ばれるのか？」
4. **ツールの連携を理解する** — 「Wi-Fi 攻撃では airodump-ng → aireplay-ng → aircrack-ng がどう連携するか？」

### 特に理解を深めたいツール（優先度★★★）

| 優先度 | ツール | 理解のポイント |
|---|---|---|
| ★★★ | Nmap | スキャン種別の違いと検出原理、IDS への影響 |
| ★★★ | Metasploit | モジュール・ペイロード・セッションの概念 |
| ★★★ | Wireshark | プロトコル解析・異常トラフィックの読み方 |
| ★★★ | Burp Suite | HTTP インターセプト・脆弱性診断の流れ |
| ★★★ | Aircrack-ng | 無線LAN の認証フロー・ハンドシェイクの仕組み |
| ★★☆ | Hashcat / John | ハッシュ関数・攻撃手法の種類と防御策 |
| ★★☆ | Hydra | オンライン総当たりのリスクと対策（アカウントロック等） |
| ★★☆ | Nessus | CVSS スコアの読み方・脆弱性管理サイクル |
| ★★☆ | Maltego | OSINT の情報源と組織露出リスクの評価 |
| ★★☆ | SET | ソーシャルエンジニアリングの手口と社員教育での活用 |

---

## ツール分類の早見表

| カテゴリ | 代表ツール |
|---|---|
| OSINT | Maltego, theHarvester, Recon-ng, Shodan |
| ポートスキャン | Nmap, Masscan, Angry IP Scanner |
| 脆弱性スキャン | Nessus, OpenVAS, Nexpose |
| 列挙 | Enum4linux, Gobuster, SNMPwalk |
| エクスプロイト | Metasploit, Exploit-DB |
| パスワードクラック | Hashcat, John, Hydra, Aircrack-ng |
| Webアプリ | Burp Suite, SQLmap, Nikto, ZAP |
| ソーシャルエンジニアリング | SET, Gophish |
| 無線LAN | Aircrack-ng, Kismet, Wifite |
| スニッフィング/MITM | Wireshark, Ettercap, Bettercap, Responder |
| 持続化/RAT | Netcat, Meterpreter, Cobalt Strike |
| 権限昇格 | Mimikatz, LinPEAS, PowerSploit |
| IDS 回避 | Proxychains, Veil, Tor |
| フォレンジック | Autopsy, Volatility, FTK |
| ステガノグラフィ | Steghide, Stegdetect |
| DoS | LOIC, hping3, Slowloris |

---

## まとめ

レッドチームセキュリティの学習では、「**どのフェーズで**・**何の目的で**・**どのツールを使うか**」を体系的に把握することが出発点になります。

- **Nmap, Metasploit, Wireshark, Burp Suite, Aircrack-ng** は特に理解を深める価値のある基本ツール
- 各ツールの動作原理・コマンドと、それに対する**防御策・検知方法**をセットで学ぶ
- ツールを「攻撃5フェーズのどこに位置するか」でマッピングすると全体像が掴みやすい

攻撃側の手法を学ぶ目的は、より強固な防御を設計することにあります。得た知識は**倫理的・合法的な範囲**でのみ活用してください。

:::message alert
本記事の内容を**無許可の環境に適用することは違法**です。学習・研究・CTF・合法的なペネトレーションテスト以外への転用は絶対にしないでください。
:::

---

## 参考

- [Nmap 公式ドキュメント](https://nmap.org/book/man.html)
- [Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
