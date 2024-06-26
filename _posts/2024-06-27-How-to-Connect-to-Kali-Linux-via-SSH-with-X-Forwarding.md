---
layout: post
title: How to Connect to Kali Linux via SSH with X Forwarding
date: 2024-06-27 01:35
category: [Linux]
author: 
tags: [Linux, SSH]
summary: 
img_path: /assets/img/screenshots
image:
  path: Xquartz.png
  width: 100%
  height: 100%
  alt: Xquartz
---

# Kali LinuxへSSHでX接続する方法

SSHでX接続を行うことで、リモートのKali Linuxマシン上でGUIアプリケーションを使用することができます。以下にその手順を説明します。

## 必要な準備

1. **SSHサーバの設定**:
   - Kali Linux上でSSHサーバがインストールされ、実行されていることを確認します。
     ```bash
     sudo apt update
     sudo apt install openssh-server
     sudo systemctl start ssh
     sudo systemctl enable ssh
     ```

2. **X11Forwardingの有効化**:
   - SSHサーバの設定ファイルを編集して、X11転送を有効にします。
     ```bash
     sudo nano /etc/ssh/sshd_config
     ```
   - 次の行を探し、コメントを外して`yes`に設定します。
     ```plaintext
     X11Forwarding yes
     ```
   - 設定を保存して、SSHサーバを再起動します。
     ```bash
     sudo systemctl restart ssh
     ```

3. **ローカルマシンの準備**:
   - ローカルマシンがLinuxであれば、必要なXサーバがインストールされていることを確認します。
   - Windowsの場合、Xサーバ（例：XmingやVcXsrv）をインストールします。
   - Macの場合、XQuartzをインストールします。

## X11転送を使用してSSH接続を行う手順

1. **SSH接続を行う**:
   - ローカルマシンからKali Linuxマシンに対して、`-X`オプションを使用してSSH接続を行います。
     ```bash
     ssh -X username@kali_ip_address
     ```
   - `username`はKali Linuxのユーザー名、`kali_ip_address`はKali LinuxマシンのIPアドレスです。

2. **GUIアプリケーションの実行**:
   - SSH接続後、Kali Linux上でGUIアプリケーションを実行します。
     ```bash
     xclock
     ```
   - これにより、ローカルマシン上にxclockのウィンドウが表示されれば、X11転送が正しく機能していることが確認できます。

## トラブルシューティング

- **X11転送が機能しない場合**:
  - `/etc/ssh/sshd_config`で`X11Forwarding yes`が正しく設定されているか確認してください。
  - ローカルマシン上でXサーバが実行されているか確認してください。
  - 必要に応じて、SSH接続時に`-Y`オプションを使用してみてください。
    ```bash
    ssh -Y username@kali_ip_address
    ```

この手順を踏めば、SSHを通じてKali Linux上のGUIアプリケーションをローカルマシンで使用することができます。


