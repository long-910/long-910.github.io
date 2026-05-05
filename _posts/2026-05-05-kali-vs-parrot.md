---
layout: post
title: "Kali Linux vs Parrot OS ― セキュリティ特化ディストリビューションを徹底比較"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["linux", "security", "kalilinux", "parrotos", "pentest"]
date: "2026-05-05 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-05-kali-vs-parrot)でも公開しています。


## はじめに

ペネトレーションテストや脆弱性調査に使うLinuxディストリビューションとして真っ先に名前が挙がるのが **Kali Linux** と **Parrot OS** です。どちらもDebian系で、セキュリティツールが多数プリインストールされていますが、設計思想・ターゲットユーザー・パフォーマンスなど、多くの点で異なります。

この記事は、自分なりに調べた内容を備忘録としてまとめたものです。

:::message alert
本記事は個人が調査・検証した内容をもとにまとめたものです。記載している仕様・数値・ツール情報などは実際の環境や時期によって異なる場合があり、事実と異なる記述が含まれている可能性があります。正確な情報は各ディストリビューションの公式サイトやドキュメントをご確認ください。
:::

---

## 1. 両ディストリビューションの概要

### Kali Linux

| 項目 | 内容 |
|---|---|
| **開発元** | Offensive Security |
| **ベース** | Debian Testing |
| **初リリース** | 2013年（前身のBackTrackは2006年） |
| **最新版** | Kali Linux 2026.1（2026年3月） |
| **公式サイト** | kali.org |

Kali Linuxは**プロフェッショナルのペネトレーションテスター向け**に設計されたディストリビューションです。前身のBackTrack Linuxを一から再設計し、2013年にリリースされました。Offensive Securityが管理・開発し、OSCP（Offensive Security Certified Professional）などの業界認定資格と強く結びついています。

セキュリティツールのみに特化した「職人の道具箱」的な位置づけで、日常的な使用よりも**特定の作業環境で使うことを前提**としています。

### Parrot OS

| 項目 | 内容 |
|---|---|
| **開発元** | Frozenbox（コミュニティ主導） |
| **ベース** | Debian Stable |
| **初リリース** | 2013年 |
| **最新版** | Parrot OS 7.0（2025年12月） |
| **公式サイト** | parrotsec.org |

Parrot OSは**セキュリティとプライバシーと日常利用を両立**させることをコンセプトにしています。複数のエディションを提供しており、セキュリティ作業専用の「Security Edition」だけでなく、ツールなしの「Home Edition」もあります。

Kaliとは異なり、**日常的なデスクトップOSとしても使える**設計になっており、初心者から中級者まで幅広いユーザーを対象にしています。

---

## 2. エディション構成の違い

### Kali Linux のエディション

```
Kali Linux
├── Installer          # 標準インストーラー版
├── Live               # インストール不要のライブ起動版
├── NetInstaller       # ネット越しにパッケージを取得するミニマル版
├── WSL                # Windows Subsystem for Linux 向け
├── Cloud              # AWS・Azure・GCP 向けイメージ
├── Containers         # Docker/LXC/LXD コンテナ版
├── Virtual Machines   # VMware/VirtualBox 向けプリビルド
├── ARM                # Raspberry Pi などの ARM 端末向け
└── Kali Purple        # 防御側（ブルーチーム）向け特別エディション（2023年〜）
```

**Kali Purple** は2023年に導入されたユニークなエディションで、攻撃ツールに加えてSIEM（Splunk・ELK）や脅威インテリジェンスツールなど**防御側のツール**も含んでいます。

### Parrot OS のエディション

```
Parrot OS
├── Security Edition   # フルのセキュリティ・ペンテストツール付き
├── Home Edition       # セキュリティツールなしの日常利用版
└── HTB Edition        # Hack The Box 向けに最適化されたエディション
```

Parrot OSの大きな特徴は**Home Editionの存在**です。セキュリティツールを一切含まない軽量なデスクトップOSとして使え、必要なツールだけを後からインストールするスタイルが取れます。

---

## 3. システム要件の比較

### Kali Linux の要件

| 項目 | 最小 | 推奨 |
|---|---|---|
| **RAM** | 512 MB（SSHのみなら128 MB） | 4〜8 GB |
| **ストレージ** | 20 GB | 50 GB 以上 |
| **CPU** | 2 GHz以上（32/64bit） | 64bit マルチコア |
| **ネット** | ― | インストール時は推奨 |

### Parrot OS の要件

| 項目 | 最小 | 推奨 |
|---|---|---|
| **RAM** | 256 MB（Home） / 2 GB（Security） | 4 GB 以上 |
| **ストレージ** | 20 GB（Home） / 40 GB（Security） | 50 GB 以上 |
| **CPU** | 64bit デュアルコア | 64bit マルチコア |
| **アーキテクチャ** | x86_64, ARM | ―（7.0でRISC-V追加） |

**ポイント:** Parrot OS は特に RAM 効率が優れており、**2 GB の RAM でも快適に動作**します。古いノートPCやRaspberry Piでの利用に向いています。Kaliは現代的なハードウェアを前提に設計されているため、低スペック環境では動作が重くなることがあります。

---

## 4. デスクトップ環境

### Kali Linux

| バージョン | デフォルトDE | 選択可能なDE |
|---|---|---|
| 2019.4〜現在 | **Xfce** | GNOME, KDE Plasma, i3, Sway など |

Kali 2025〜2026では Wayland サポートが強化され、GNOME・KDE・Xfce すべてで品質が向上しています。2026.1ではリリース20周年（BackTrack時代から）を記念した**テーマ刷新**が行われ、新しいブートアニメーション・インストーラー・デスクトップテーマが導入されました。

```bash
# Kali でデスクトップ環境を切り替えるには（例：GNOMEをインストール）
sudo apt install kali-desktop-gnome
# ログイン時にセッションを選択する
```

### Parrot OS

| バージョン | デフォルトDE | 選択可能なDE |
|---|---|---|
| 〜6.x | **MATE** | XFCE |
| 7.0〜 | **KDE Plasma** | MATE（引き続きサポート） |

Parrot OS 7.0 では Debian 13「Trixie」をベースにしてデフォルトのデスクトップを **KDE Plasma に変更**しました。MATE は引き続き提供されますが、メインのデスクトップはKDEに移行しています。

MATE時代のParrot OSはRAM消費が256〜320 MBと非常に軽量でしたが、KDE移行後も依然としてKaliより軽量に設計されています。

---

## 5. プリインストールツールの比較

### Kali Linux のツール構成（600種類以上）

Kaliのツールはセキュリティ作業のワークフローに沿ってカテゴリ分けされています。

| カテゴリ | 代表的なツール |
|---|---|
| **情報収集** | Nmap, Maltego, theHarvester, Recon-ng |
| **脆弱性スキャン** | OpenVAS, Nikto, Nessus（試用版） |
| **エクスプロイト** | Metasploit Framework, searchsploit |
| **Webアプリ** | Burp Suite, OWASP ZAP, SQLmap, wfuzz |
| **パスワード攻撃** | John the Ripper, Hashcat, Hydra, Medusa |
| **無線攻撃** | Aircrack-ng, Kismet, Bettercap |
| **フォレンジクス** | Autopsy, Binwalk, Volatility |
| **スニッフィング** | Wireshark, tcpdump, Ettercap |
| **リバースエンジニアリング** | Ghidra, radare2, GDB |
| **ソーシャルエンジニアリング** | SET（Social-Engineer Toolkit） |

```bash
# Kali でツール一覧を確認する
kali-tools list

# カテゴリ別にインストール（例：Webアプリテスト系）
sudo apt install kali-tools-web

# 全ツールをインストール（ディスク容量に注意：50 GB+）
sudo apt install kali-linux-everything
```

### Parrot OS のツール構成（Security Edition：700種類以上）

Parrot OS Security Edition は Kali と同等のツール群を持ちつつ、**プライバシーツールが充実**しています。

| カテゴリ | 代表的なツール・特徴 |
|---|---|
| **ペンテスト系** | Metasploit, Nmap, Burp Suite（Kaliと同等） |
| **プライバシー** | AnonSurf（Torルーティング）, Tor Browser, OnionShare |
| **暗号化** | VeraCrypt, GnuPG, KeePassXC |
| **C2フレームワーク** | Sliver C2（2025年追加）, Empire 6.1.2 |
| **Webプロキシ** | Caido（2025年追加） |
| **ブラウザ拡張** | uBlock Origin, Privacy Badger, HTTPS Everywhere（プリ設定済み） |

**AnonSurf** はParrot OS独自のツールで、システム全体のトラフィックをTorネットワーク経由にルーティングします。

```bash
# AnonSurf の使い方（Parrot OS）
sudo anonsurf start    # Torルーティング開始
sudo anonsurf status   # 現在のステータス確認
sudo anonsurf stop     # ルーティング停止
sudo anonsurf myip     # 現在のIPアドレス確認
```

---

## 6. パフォーマンス比較

実際の使用感として、以下のような傾向があります。

### 起動時間（目安）

| 環境 | Kali Linux | Parrot OS |
|---|---|---|
| **SSD（NVMe）搭載機** | 約15〜20秒 | 約10〜15秒 |
| **HDD搭載機** | 約40〜60秒 | 約25〜40秒 |
| **4 GB RAM の VM** | 約25〜35秒 | 約20〜28秒 |

### RAM 使用量（アイドル時・目安）

| 環境 | Kali Linux（Xfce） | Parrot OS（MATE/KDE） |
|---|---|---|
| **最小起動直後** | 約600〜800 MB | 約350〜500 MB |
| **ブラウザ1タブ開いた状態** | 約1.2〜1.5 GB | 約800 MB〜1.2 GB |

**ポイント:** 低スペックマシンではParrot OSが明確に有利です。一方、高スペックな専用マシンや仮想環境であれば、Kaliのパフォーマンスで困ることはほぼありません。

---

## 7. セキュリティ設定の違い

### Kali Linux のデフォルト設定

Kali は**セキュリティより利便性を優先**したデフォルト設定になっています。

```bash
# Kali のデフォルト設定の特徴
- root ログインが設定可能（2020年以前はrootがデフォルト）
- SSH サーバーは無効（セキュリティのため）
- ファイアウォール（ufw）は未設定
- Kali Undercover モード（Windows 10風UIに瞬時に変換できる機能）
```

2020年以前のKaliはrootユーザーがデフォルトでしたが、現在は通常ユーザー（kali/kali）での運用が標準となっています。

### Parrot OS のデフォルト設定

Parrot は**プライバシーと匿名性を重視**したデフォルト設定です。

```bash
# Parrot のデフォルト設定の特徴
- 通常ユーザーでの運用がデフォルト（sudo使用）
- AnonSurf によるTorルーティングが即時使用可能
- Firefox にプライバシー拡張が事前設定済み
- 定期的なセキュリティ監査を経たパッケージ管理
```

---

## 8. 学習リソースとコミュニティ

### Kali Linux

| リソース | 詳細 |
|---|---|
| **公式ドキュメント** | docs.kali.org（非常に充実） |
| **公式トレーニング** | kali.training（無料〜有料コース） |
| **認定資格** | OSCP, OSWP, OSED など Offensive Security 系 |
| **コミュニティ** | Reddit(r/Kalilinux), Discord, IRC |
| **YouTube** | 豊富な解説動画（英語・日本語） |

Kaliは**Offensive Securityという企業が支援**しているため、公式ドキュメントやトレーニング素材が充実しています。OSCPなどの業界認定資格との親和性が高く、体系的に学べる環境が整っています。

### Parrot OS

| リソース | 詳細 |
|---|---|
| **公式ドキュメント** | parrotsec.org/docs/（整備中） |
| **コミュニティフォーラム** | community.parrotsec.org |
| **GitLab** | コントリビューション歓迎 |
| **HTB Edition** | Hack The Box との公式コラボ |
| **コミュニティ** | IRC, Matrix, Discord |

Parrot OSは**コミュニティ主導**のプロジェクトであるため、Kaliと比べてドキュメントや学習リソースが少ないのが現状です。ただし、HTB（Hack The Box）との公式コラボエディションがあり、CTFや実践学習には適しています。

---

## 9. 最新バージョン情報（2026年5月時点）

### Kali Linux 2026.1（2026年3月リリース）

- **カーネル:** Linux 6.18
- **テーマ刷新:** 20周年記念の大規模ビジュアルオーバーホール
- **BackTrack Mode:** 懐かしのBackTrack 5 UIを再現できるノスタルジックモード
- **新ツール:** 8つの新規ツールを追加（合計600種類以上）
- **NetHunter:** WPSスキャンのバグ修正、Samsung S10サポート
- **デスクトップ:** GNOME・KDE・Xfce すべて品質向上

```bash
# Kali を最新版に更新
sudo apt update && sudo apt full-upgrade -y
```

### Parrot OS 7.0（2025年12月リリース）

- **ベース:** Debian 13「Trixie」
- **デフォルトDE:** KDE Plasma（MATEから変更）
- **アーキテクチャ:** RISC-V サポート追加
- **新ツール:** Sliver C2, Caido, Empire 6.1.2, PowerShell 7.5, .NET ランタイム（5〜9）
- **方向性:** クラウド・コンテナ向けの軽量デプロイメントに注力

### Parrot OS 6.4（2025年7月リリース・6.x系最終版）

- **カーネル:** Linux 6.12.32 LTS
- **Metasploit:** 6.4.71（最新版）
- **Firefox:** プライバシーパッチ適用済み LTS 版

---

## 10. どちらを選ぶべきか

### Kali Linux を選ぶべきシーン

```
✅ 業界認定資格（OSCP, CEH, GPEN など）の取得を目指している
✅ プロのペネトレーションテスターとして企業で働いている
✅ 4 GB 以上の RAM を持つ専用マシンまたはVM環境がある
✅ 豊富なドキュメントと大きなコミュニティを重視する
✅ セキュリティ特化の環境で最大限のツールを使いたい
✅ 無線攻撃・フォレンジクスなど特定分野のツールを重点的に使う
✅ Offensive Security のエコシステム（Exploit DB など）と連携したい
```

### Parrot OS を選ぶべきシーン

```
✅ セキュリティ学習を始めたばかりの初心者・学生
✅ RAM が 2〜4 GB の古いマシンやラップトップを使っている
✅ 日常のデスクトップ作業とセキュリティ作業を1台でこなしたい
✅ プライバシーと匿名性（Tor, AnonSurf）を重視する
✅ CTF（Capture The Flag）やHack The Boxでの学習が主な目的
✅ 軽量で省エネな環境を求めている
✅ コンテナ・クラウド環境でのセキュリティ作業をしたい（7.0以降）
```

### 判断フローチャート

```
セキュリティLinuxを選びたい
        │
        ▼
   ハードウェアのスペックは？
   ┌──────────────────────┐
   │                      │
  低スペック             高スペック
（RAM 4 GB 未満）      （RAM 4 GB 以上）
   │                      │
   ▼                      ▼
 Parrot OS          プロ資格・仕事目的？
                    ┌──────────┐
                    │          │
                   YES         NO
                    │          │
                    ▼          ▼
                Kali Linux  Parrot OS
                            （学習・日常利用）
```

---

## 11. 両方を共存させる方法

実際のセキュリティプロフェッショナルは、Kali と Parrot を**用途に応じて使い分ける**ことが多いです。

### 仮想マシンで複数環境を管理する

```bash
# VirtualBox でスナップショットを活用した管理例
VirtualBox/
├── Kali-2026.1/          # プロの作業環境（フルツール）
│   ├── snapshot-clean    # クリーンな状態のスナップショット
│   └── snapshot-working  # 作業中の状態
└── Parrot-7.0/           # 軽量・プライバシー重視環境
    ├── snapshot-clean
    └── snapshot-ctf      # CTF演習用
```

### Docker でツールを管理する

```bash
# Kali Linux の Docker イメージを使う
docker pull kalilinux/kali-rolling
docker run -it --rm kalilinux/kali-rolling /bin/bash

# Parrot OS の Docker イメージを使う
docker pull parrotsec/security
docker run -it --rm parrotsec/security /bin/bash
```

---

## まとめ

| 比較項目 | Kali Linux | Parrot OS |
|---|---|---|
| **ターゲット** | プロ・上級者 | 初心者〜中級者 |
| **最低RAM** | 512 MB（実用4 GB+） | 256 MB（実用2 GB+） |
| **デフォルトDE** | Xfce | KDE Plasma（7.0〜） |
| **ツール数** | 600種類以上 | 700種類以上（Security版） |
| **プライバシー** | 基本的 | 高（AnonSurf, Tor内蔵） |
| **パフォーマンス** | 高スペックで強い | 低スペックでも快適 |
| **コミュニティ** | 大きく充実 | 小さいが成長中 |
| **日常利用** | 向かない | Home版で可能 |
| **資格連携** | OSCP など強力 | HTB Editionあり |
| **最新カーネル** | Linux 6.18 | Linux 6.12 LTS |

「どちらが優れているか」という問いに対する答えは**ユースケース次第**です。

- **Kaliは「業界標準の道具箱」** ― プロフェッショナルなペンテスト作業に特化し、資格やドキュメントが充実
- **ParrotOSは「万能な相棒」** ― 軽量で日常利用からセキュリティ作業まで対応し、プライバシーを重視

初学者であればParrot OSのHome EditionかSecurity Editionから入るのが負担が少なくおすすめです。キャリアとしてセキュリティを目指すなら、Kali Linuxに慣れ親しんでおくことが就職後に役立ちます。

どちらも無料でダウンロードできるため、まずは仮想マシンで両方を試してみることをおすすめします。

---

## 参考リンク

- [Kali Linux 公式サイト](https://www.kali.org/)
- [Kali Linux 2026.1 リリースノート](https://www.kali.org/blog/kali-linux-2026-1-release/)
- [Parrot Security 公式サイト](https://www.parrotsec.org/)
- [Parrot OS 7.0 リリースノート](https://parrotsec.org/blog/parrot-7.0-release-notes/)
- [Offensive Security トレーニング](https://www.offsec.com/courses-and-certifications/)
- [Hack The Box - Parrot HTB Edition](https://www.hackthebox.com/)
