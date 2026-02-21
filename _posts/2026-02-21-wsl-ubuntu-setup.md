---
layout: post
title: "Windows WSL入門：インストールからUbuntu環境構築まで完全ガイド"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["wsl", "ubuntu", "windows", "linux", "環境構築"]
date: "2026-02-21 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-02-21-wsl-ubuntu-setup)でも公開しています。

# Windows WSL入門：インストールからUbuntu環境構築まで完全ガイド

## はじめに

WSL（Windows Subsystem for Linux）は、Windows上でLinux環境をネイティブに動作させる仕組みです。仮想マシンと異なり、オーバーヘッドが少なく、Windowsとのファイル共有も容易です。本記事では、WSLのインストールからUbuntu環境の各種設定まで、備忘録として詳しくまとめます。

## WSLとは

WSL（Windows Subsystem for Linux）には現在2つのバージョンがあります。

| 項目 | WSL1 | WSL2 |
|------|------|------|
| アーキテクチャ | システムコール変換 | 軽量仮想マシン（Hyper-V） |
| Linux互換性 | 部分的 | ほぼ完全 |
| ファイルI/O速度 | Windowsファイル：速い | Linuxファイル：速い |
| ネットワーク | Windowsと共有 | 独立したNIC |
| Docker対応 | 非推奨 | 対応 |

**推奨：WSL2**を使用します。

## 前提条件

- Windows 10 バージョン 2004 以降（ビルド 19041 以降）
- Windows 11（すべてのバージョン）
- BIOSで仮想化機能（VT-x/AMD-V）が有効

## WSLのインストール

### 方法1：コマンド一発インストール（推奨）

PowerShellまたはコマンドプロンプトを**管理者権限**で起動し、以下を実行します。

```powershell
wsl --install
```

このコマンドで以下が自動で行われます：
- WSL機能の有効化
- 仮想マシンプラットフォームの有効化
- Linuxカーネルのインストール
- WSL2をデフォルトに設定
- Ubuntuのインストール

完了後、**PCを再起動**します。

### 方法2：特定のディストリビューションを指定してインストール

```powershell
# 利用可能なディストリビューションの一覧
wsl --list --online

# Ubuntu 22.04 LTSをインストール
wsl --install -d Ubuntu-22.04

# Ubuntu 24.04 LTSをインストール
wsl --install -d Ubuntu-24.04
```

### WSLのバージョン確認と設定

```powershell
# インストール済みディストリビューションの確認
wsl --list --verbose

# WSL2をデフォルトに設定
wsl --set-default-version 2

# 特定のディストリビューションをWSL2に変換
wsl --set-version Ubuntu 2
```

## Ubuntuの初期設定

再起動後、Ubuntuを起動するとユーザー名とパスワードの設定を求められます。

```
Enter new UNIX username: your_username
New password: ********
Retype new password: ********
```

:::message
このユーザー名とパスワードはWindowsのアカウントとは独立しています。sudoコマンド実行時に必要になるので必ず覚えておきましょう。
:::

## パッケージの更新

まず最初にパッケージリストとパッケージ自体を更新します。

```bash
sudo apt update && sudo apt upgrade -y
```

## 日本語環境の設定

### 日本語パッケージのインストール

```bash
sudo apt install -y language-pack-ja
sudo update-locale LANG=ja_JP.UTF-8
```

### ロケールの確認

```bash
locale
```

以下のように表示されれば設定完了です。

```
LANG=ja_JP.UTF-8
LANGUAGE=
LC_CTYPE="ja_JP.UTF-8"
...
```

### タイムゾーンの設定

```bash
sudo timedatectl set-timezone Asia/Tokyo

# 確認
timedatectl
```

## 開発ツールのインストール

### 基本的なビルドツール

```bash
sudo apt install -y build-essential
```

`build-essential`には以下が含まれます：
- gcc（Cコンパイラ）
- g++（C++コンパイラ）
- make
- その他開発に必要なパッケージ

### Git

```bash
sudo apt install -y git

# バージョン確認
git --version

# 基本設定
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global core.autocrlf input
```

### curl / wget

```bash
sudo apt install -y curl wget
```

### Node.js（nvmを使用）

```bash
# nvmのインストール
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# シェルを再読み込み
source ~/.bashrc

# Node.js LTSのインストール
nvm install --lts

# バージョン確認
node --version
npm --version
```

### Python

```bash
# Python3とpipのインストール
sudo apt install -y python3 python3-pip python3-venv

# バージョン確認
python3 --version
pip3 --version
```

## シェルの設定

### Zshのインストールとデフォルト化

```bash
sudo apt install -y zsh

# デフォルトシェルを変更
chsh -s $(which zsh)
```

### Oh My Zshのインストール（オプション）

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## ファイルシステムの理解

WSLとWindowsのファイルシステムのアクセス方法を理解することが重要です。

### WindowsからWSLのファイルへアクセス

エクスプローラーのアドレスバーに以下を入力します。

```
\\wsl$\Ubuntu
```

または、WSL内から以下のコマンドでエクスプローラーを開けます。

```bash
explorer.exe .
```

### WSLからWindowsのファイルへアクセス

Windowsのドライブは`/mnt/`以下にマウントされています。

```bash
# Cドライブへのアクセス
ls /mnt/c/

# Windowsのデスクトップへ移動
cd /mnt/c/Users/ユーザー名/Desktop
```

:::message alert
パフォーマンスの観点から、Linuxファイルの操作は`/home/`以下で行うことを推奨します。`/mnt/c/`以下でのLinux操作は速度が低下します。
:::

## WSLの設定ファイル

### `.wslconfig`（Windowsユーザーフォルダに配置）

`C:\Users\ユーザー名\.wslconfig`を作成して、WSL2全体のリソース設定を行います。

```ini
[wsl2]
# メモリ上限（物理メモリの50%程度が目安）
memory=4GB

# CPU数
processors=4

# スワップ領域
swap=2GB

# localhostへの転送
localhostForwarding=true
```

### `/etc/wsl.conf`（Ubuntu内に配置）

Ubuntu内の`/etc/wsl.conf`で、ディストリビューション固有の設定を行います。

```ini
[boot]
# systemdを有効化（Ubuntu 22.04以降）
systemd=true

[automount]
# Windowsドライブの自動マウント
enabled=true
# マウントオプション
options="metadata,umask=22,fmask=11"

[network]
# ホスト名の設定
hostname=my-wsl

[user]
# デフォルトユーザー
default=your_username
```

設定変更後はWSLを再起動します。

```powershell
# PowerShellで実行
wsl --shutdown
```

## Visual Studio Codeとの連携

### WSL拡張機能のインストール

VSCode側で`WSL`拡張機能をインストールします（拡張機能ID: `ms-vscode-remote.remote-wsl`）。

### WSLからVSCodeを起動

WSLのターミナルから以下のコマンドでVSCodeを起動できます。

```bash
# カレントディレクトリをVSCodeで開く
code .
```

初回起動時はVSCode Serverが自動でインストールされます。

## Dockerとの連携

### Docker Desktopを使用する方法（推奨）

1. [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)をインストール
2. Docker Desktopの設定で「Use the WSL 2 based engine」を有効化
3. 「WSL Integration」でUbuntuを選択して有効化

```bash
# WSL内でDockerが使えることを確認
docker --version
docker run hello-world
```

## よく使うコマンド

### WSL管理コマンド（PowerShellで実行）

```powershell
# WSLの起動
wsl

# 特定のディストリビューションを起動
wsl -d Ubuntu

# WSLのシャットダウン
wsl --shutdown

# 特定のディストリビューションを終了
wsl --terminate Ubuntu

# ディストリビューションのエクスポート（バックアップ）
wsl --export Ubuntu ubuntu-backup.tar

# ディストリビューションのインポート（復元）
wsl --import Ubuntu C:\wsl\ubuntu ubuntu-backup.tar

# ディストリビューションの削除
wsl --unregister Ubuntu
```

## トラブルシューティング

### WSLが起動しない場合

```powershell
# Windows機能の確認・有効化
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

再起動後、[Linuxカーネル更新プログラム](https://aka.ms/wsl2kernel)をインストールします。

### ファイルのパーミッションが正しくない場合

`/etc/wsl.conf`に以下を追加します。

```ini
[automount]
options="metadata"
```

### WSLのネットワークが繋がらない場合

```bash
# DNSの設定確認
cat /etc/resolv.conf

# 手動でDNSを設定（/etc/wsl.confで設定した場合）
```

`/etc/wsl.conf`に以下を追加します。

```ini
[network]
generateResolvConf=false
```

その後`/etc/resolv.conf`を編集します。

```bash
sudo nano /etc/resolv.conf
# nameserver 8.8.8.8 を追加
```

### メモリが解放されない場合

WSL2はメモリを積極的にキャッシュするため、Windowsタスクマネージャーで大量のメモリを消費しているように見えることがあります。

```powershell
# WSLを再起動してメモリを解放
wsl --shutdown
```

または`.wslconfig`でメモリ上限を設定します。

## まとめ

WSLを使うことで、Windows上でLinuxの豊富なツールをほぼネイティブな速度で利用できます。特に開発環境として非常に優秀です。本記事で紹介した設定を行えば、快適な開発環境が整います。

## 参考リンク

- [Microsoft公式：WSLのインストール](https://learn.microsoft.com/ja-jp/windows/wsl/install)
- [Microsoft公式：WSLの詳細設定の構成](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config)
- [Microsoft公式：WSLでのファイルの権限](https://learn.microsoft.com/ja-jp/windows/wsl/file-permissions)
- [Ubuntu公式ドキュメント](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support)
