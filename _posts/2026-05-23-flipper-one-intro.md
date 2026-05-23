---
layout: post
title: "Flipper One が気になったので調べてみた — ポケットサイズのLinux ARMコンピュータ"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["flipper", "linux", "security", "hardware", "networking"]
date: "2026-05-23 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-23-flipper-one-intro)でも公開しています。


## Flipper One とは

2026年5月20日、Flipper Devices社が **Flipper One** を正式発表しました。Flipper Zeroの後継として長らく噂されていた本製品ですが、実際にはFlipperシリーズとして「別の役割」を担う新しいデバイスです。

Flipper OneはポケットサイズのARM Linuxコンピュータで、**IPネットワーク解析・高速データ処理・オンデバイスAI**に特化した設計になっています。Flipper Zero（シグナル解析・RFID・NFCなどのワイヤレスハッキングツール）とは用途が異なる「補完的な存在」として位置づけられています。

---

## ハードウェアスペック

### プロセッサ

| コンポーネント | 詳細 |
|---|---|
| メインCPU | Rockchip RK3576（オクタコア） |
| Cortex-A72 | 4コア / 最大 2.2 GHz |
| Cortex-A53 | 4コア / 最大 2.0 GHz |
| GPU | Mali-G52 |
| NPU（AI アクセラレータ） | **6 TOPS** |
| サブMCU | Raspberry Pi RP2350（2× Cortex-M33, 150 MHz） |

メインCPUのRK3576はハイパフォーマンスなARMプロセッサで、Mali-G52 GPUに加えて**6 TOPSのNPU**を内蔵しています。このNPUにより、クラウドを使わずデバイス単体でローカルLLM（大規模言語モデル）を実行できます。

RP2350サブMCUはディスプレイ・ボタン・タッチパッド・LED・電源管理を担当します。

### ディスプレイ

- **2.4インチ カラー液晶**（256 × 144ピクセル）
- Flipper Zeroの1.4インチモノクロ液晶から大幅刷新

### サイズ

- **長さ 152.6 mm × 高さ 67 mm × 厚さ 40 mm**
- Flipper Zero（97.5 mm）より一回り大きい

---

## ネットワーク・接続性

Flipper Oneの最大の特徴のひとつが、充実したネットワーク機能です。

| インターフェース | 仕様 |
|---|---|
| Ethernet | **2× Gigabit Ethernet** |
| USB Ethernet | USB 3.0（5 Gbps） |
| Wi-Fi | **Wi-Fi 6E**（2.4 / 5 / 6 GHz対応） |
| モバイル通信 | M.2スロット経由で**5G / 4G LTE**対応（オプション） |

デュアルGigabit EthernetにWi-Fi 6Eまで搭載されており、**ポータブルなネットワーク解析機器**としても本格的に使えます。現場でパケットキャプチャやトラフィック分析を行うセキュリティエンジニアにとって魅力的なスペックです。

---

## 拡張性

### M.2スロット

- 対応フォームファクタ：**2242・3042・3052**
- 5G/4G LTEモデム・NVMe SSDなどを搭載可能
- Flipper Zeroで標準搭載だったサブGHz・NFC・RFIDはここで追加する形

### GPIOヘッダー（背面）

- **20ピンGPIO**（CPU＋MCUからのピンを集約）
- USB 2.0、5V、3.3V、GND
- 14ピン デバッグポート
- ピッチは2.54mm（標準ユニバーサル基板と同じ）なのでDIYモジュール自作が容易

---

## OS：Debian 13 Linux

Flipper OneはフルのLinuxディストリビューションである**Debian 13**を搭載。これにより以下のようなプロフェッショナルツールをそのまま使えます。

- `tcpdump` / `Wireshark` — パケットキャプチャ
- `nftables` / `iptables` — ファイアウォール・パケットフィルタリング
- `nmap` — ネットワークスキャン
- `Metasploit` などのセキュリティツール（別途インストール）
- `ollama` などのローカルLLMランタイム（NPU活用）

Debianベースなので、aptで任意のパッケージをインストールでき、汎用Linuxマシンとして幅広い用途に対応します。

---

## 何ができるのか：主なユースケース

### 1. ポータブルネットワーク解析

デュアルGigabit EthernetとWi-Fi 6EでLANに接続し、現場でパケットキャプチャやトラフィック分析が可能。`tcpdump`や`Wireshark`をそのまま動かせます。ルータやスイッチのミラーポートに接続して**インラインで通信を観察する**使い方が想定されています。

### 2. ハードウェアハッキング・電子工作

20ピンGPIOヘッダーでマイコン・センサー・電子部品と直接接続できます。M.2スロットに専用モジュールを挿せばサブGHz無線・NFC・RFIDにも対応。**Flipper Zeroにできたことを上位CPUでより深く行う**ことも可能です。

### 3. オンデバイスAI（ローカルLLM）

6 TOPSのNPUを活用して、`ollama`などでLLMをオフラインで実行できます。クラウドなしでプライバシーを確保したAIアシスタントを現場で使うといった活用が期待されます。

### 4. フィールドサイバーセキュリティ診断

Debian Linuxをベースにした診断環境として、ペネトレーションテストやネットワーク診断をポケットから実施できます。5G/4G LTE対応のM.2モジュールを挿せばインターネット接続もモバイルで確保できます。

### 5. ポータブルLinuxミニPC

開発・デバッグ・学習用のポケットサイズLinux PCとしても機能します。Wi-Fi・Ethernet・USB・GPIOと物理インターフェースが揃っており、IoT機器の開発や教育用途にも適しています。

---

## Flipper Zero との比較

Flipper Oneは「Flipper Zeroの置き換え」ではなく「別カテゴリの製品」です。

| 特徴 | Flipper Zero | Flipper One |
|---|---|---|
| 対象ユーザー | 入門者・ホビイスト | エンジニア・開発者 |
| OS | 独自RTOS | Debian 13 Linux |
| CPU | STM32WB55 | Rockchip RK3576 |
| ディスプレイ | 1.4インチ モノクロ | 2.4インチ カラー |
| Wi-Fi | なし（GPIO拡張で追加） | Wi-Fi 6E 内蔵 |
| Ethernet | なし | デュアルGigabit |
| サブGHz・NFC・RFID | 内蔵 | M.2モジュールで追加 |
| サイズ | 97.5 mm | 152.6 mm |
| 想定価格 | 約$169 | **$350以下**（予定） |
| 発売状況 | 発売中 | **未発売（開発中）** |

---

## 現在の状況：コミュニティ参加を募集中

2026年5月時点で、Flipper Oneは**まだ販売されていません**。現在はフル機能のプロトタイプ段階です。

Flipper Devicesは通常の製品発売ではなく、**プロジェクトをday-1からオープンに公開し、コミュニティの協力を求める**という珍しいアプローチをとっています。設計ファイル・ドキュメント・仕様がすべて公開されており、開発参加・フィードバック・テストを広く受け付けています。

- 発売予定時期：早くとも2026年夏以降
- 目標価格：$350以下（セルラーモジュールなし）
- 公式ドキュメント：[docs.flipper.net/one](https://docs.flipper.net/one/general/tech-specs)
- 公式ブログ：[blog.flipper.net](https://blog.flipper.net/flipper-one-we-need-your-help/)

---

## まとめ

Flipper Oneは、Flipper Zeroとは別物の**ポケットサイズARMフルLinuxコンピュータ**です。強力なRK3576プロセッサ・Wi-Fi 6E・デュアルGigabit Ethernet・6 TOPS NPU・Debian Linuxという組み合わせで、**ネットワーク解析・ハードウェアハッキング・オンデバイスAI**を一台で実現する意欲的なデバイスです。

まだ発売されていないため、購入は先になりますが、オープンな開発プロセスでコミュニティと共に進化していく点が注目されます。セキュリティエンジニア・ハードウェアハッカー・Linux愛好家にとって、今後も目が離せない製品です。

---

**参考リンク**
- [Flipper One Tech Specs - 公式ドキュメント](https://docs.flipper.net/one/general/tech-specs)
- [Flipper One — we need your help - 公式ブログ](https://blog.flipper.net/flipper-one-we-need-your-help/)
- [CNX Software: Flipper One - RK3576 ARM Linux computer](https://www.cnx-software.com/2026/05/21/flipper-one-a-rockchip-rk3576-powered-portable-arm-linux-computer-and-networking-multi-tool/)
- [TechCrunch: Flipper unveils a Linux-powered networking gadget](https://techcrunch.com/2026/05/21/flipper-unveils-a-linux-powered-networking-gadget-built-for-hackers-and-tinkerers/)
- [Tom's Hardware: Flipper One computing multitool](https://www.tomshardware.com/networking/flipper-one-computing-multitool-bristles-with-network-gpio-and-m-2-connectivity-new-keychain-device-is-also-a-fully-open-arm-linux-computer)
