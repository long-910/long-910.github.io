---
layout: post
title: "MacのUTMでParrotOSを動かしてみた（日本語入力・画面調整・共有フォルダも）"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["utm", "mac", "parrotos", "linux", "virtualization"]
date: "2026-06-02 12:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-06-02-parrot-os-utm-mac-guide)でも公開しています。


## はじめに

セキュリティ学習用に **ParrotOS** を試してみたくて、Mac 上で動かす方法を探していました。UTM を使えば無料で仮想環境が作れると聞いたので、実際にやってみました。

インストール自体は意外と簡単でしたが、**日本語入力**・**画面サイズの自動調整**・**共有フォルダの設定**などは少し手間がかかったので、その辺りも含めてまとめておきます。

:::message
環境: MacBook Air M5 / UTM v4.x / ParrotOS Security 6.x（ARM64）
:::

---

## ParrotOS って何？

**ParrotOS**（Parrot Security OS）は Debian ベースのセキュリティ特化ディストリビューションです。KaliLinux と同系統ですが、動作が比較的軽量で、日常使いもできる Home エディションもあります。

セキュリティ学習・CTF・ペネトレーションテストの練習環境としてよく使われています。

---

## 1. 必要なものを準備する

### UTM のインストール

[https://mac.getutm.app](https://mac.getutm.app) から `.dmg` をダウンロードして、アプリケーションフォルダへドラッグするだけです。

Homebrew の場合:

```bash
brew install --cask utm
```

### ParrotOS ISO のダウンロード

[ParrotOS 公式サイト](https://parrotsec.org/download/) から ISO を取ってきます。

| エディション | 説明 |
|---|---|
| **Security** | ペネトレーションテストツール一式付き |
| **Home** | 軽量・日常利用向け |

:::message
**Apple Silicon（M1〜M4）** なら **ARM64版** を選ぶのがポイントです。ARM64版を使うと UTM の「仮想化」モードで動くので、x86版よりずっと速いです。

**Intel Mac** の場合は **amd64版** を選びます。
:::

---

## 2. 仮想マシンを作る

UTM を起動して「**+**」ボタンをクリックします。

**Apple Silicon の場合（ARM64 ISO）:**
→ 「**仮想化（Virtualize）**」を選択

**Intel Mac の場合（amd64 ISO）:**
→ 「**エミュレーション（Emulate）**」を選択

次に「Linux」を選び、ダウンロードした ISO ファイルを指定します。

### ハードウェア設定

| 項目 | 設定値 |
|---|---|
| メモリ | 4096 MB（4 GB）以上がおすすめ |
| CPU コア数 | 2〜4 |
| ストレージ | 30 GB 以上（Security なら 50 GB 推奨） |

VM に名前をつけて（例: `ParrotOS-Security`）保存すれば準備完了です。

---

## 3. ParrotOS をインストールする

▶ ボタンで VM を起動すると、起動メニューが出てきます。

**「Try / Install」** を選んでデスクトップを起動し、画面上の **「Install Parrot」** アイコンをダブルクリックするとインストーラーが始まります。

### 言語・地域の設定

インストーラーで日本語を選んでおくと、インストール後もシステムが日本語表示になります。（日本語**入力**は後で別途設定します）

### パーティション

「ディスク全体を使用」を選ぶのが一番楽です。

### ユーザーアカウントの作成

ユーザー名とパスワードを設定します。このパスワードがログインや `sudo` のパスワードになります。

### インストール完了後

インストールが終わったら再起動します。**このとき ISO を取り外し忘れると、再起動後にまたインストーラーが起動してしまう**ので注意です。

1. UTM で VM を右クリック → 「編集」
2. 「ドライブ」タブ → ISO のドライブを削除
3. VM を起動

---

## 4. まず最初にパッケージを更新する

ログインしたらターミナルを開いて更新しておきます。

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 5. 日本語入力を使えるようにする

デフォルトでは日本語が入力できないので、**Fcitx5 + Mozc** をインストールします。

### インストール

```bash
sudo apt install -y \
  fcitx5 \
  fcitx5-mozc \
  fcitx5-config-qt \
  fcitx5-frontend-gtk3 \
  fcitx5-frontend-gtk4 \
  fcitx5-frontend-qt5
```

### im-config で Fcitx5 を既定の IME に設定

```bash
im-config -n fcitx5
```

### 環境変数を設定する

`~/.profile`（または `~/.xprofile`）に以下を追加します。

```bash
echo 'export GTK_IM_MODULE=fcitx' >> ~/.profile
echo 'export QT_IM_MODULE=fcitx' >> ~/.profile
echo 'export XMODIFIERS=@im=fcitx' >> ~/.profile
```

### 自動起動の設定

MATE の「**システム → 設定 → スタートアップアプリケーション**」を開き、以下を追加します。

| 項目 | 値 |
|---|---|
| 名前 | Fcitx5 |
| コマンド | `fcitx5 -d` |

### 再起動して確認

```bash
reboot
```

再起動後、タスクバーに Fcitx5 のアイコンが出ていれば OK です。テキストエディタで `半角/全角` または `Ctrl + Space` を押して日本語入力に切り替わるか確認してみてください。

---

## 6. 画面サイズを自動調整できるようにする

デフォルトだとウィンドウのサイズを変えても VM の解像度が追従しません。**SPICE ゲストエージェント** を入れると自動調整が効くようになります。

### spice-vdagent をインストール

```bash
sudo apt install -y spice-vdagent
```

### UTM 側の設定

1. UTM で VM を右クリック → 「編集」
2. 「ディスプレイ」タブ → **「動的解像度」をオン**
3. VM を再起動

これで UTM のウィンドウサイズを変えると、ParrotOS の解像度が自動で変わるようになります。

:::message
この設定をすると、Mac と VM の間でのコピー＆ペーストも使えるようになります（クリップボード共有）。
:::

---

## 7. 共有フォルダ（Mac と VM でファイルを共有）

Mac 上のフォルダを ParrotOS から直接開けるようにします。

### UTM で共有フォルダを設定

1. UTM で VM を右クリック → 「編集」
2. **「共有」タブ** → 「共有ディレクトリ」に Mac のフォルダパスを設定
3. 保存して VM を再起動

### ParrotOS 側でマウントする

マウント先のディレクトリを作って、マウントします。

```bash
sudo mkdir -p /mnt/shared
sudo mount -t 9p -o trans=virtio share /mnt/shared -o version=9p2000.L
```

`ls /mnt/shared` でファイルが見えれば成功です。

### 起動時に自動マウントさせる

毎回手動でマウントするのは面倒なので、`/etc/fstab` に追記しておきます。

```bash
sudo nano /etc/fstab
```

末尾に追加:

```
share  /mnt/shared  9p  trans=virtio,version=9p2000.L,rw,_netdev,nofail  0  0
```

### 一般ユーザーで読み書きできるようにする

デフォルトだと root 所有になることがあるので、`uid` と `gid` を指定します。

```bash
# 自分の UID/GID を確認
id
```

fstab の行を以下のように変更します（`1000` は自分の uid/gid に合わせる）:

```
share  /mnt/shared  9p  trans=virtio,version=9p2000.L,rw,_netdev,nofail,uid=1000,gid=1000  0  0
```

再起動すると自動でマウントされます。

---

## 8. パスワードを変更する

### 自分のパスワードを変更

```bash
passwd
```

現在のパスワード → 新しいパスワード → 再入力の順で入力すれば完了です。

### root のパスワードを変更

```bash
sudo passwd root
```

### GUI から変更する

**「システム → 設定 → ユーザーとグループ」** からも変更できます。GUIのほうが直感的で簡単です。

---

## 9. ハマったポイントと対処法

### 起動時に「Boot failed」と出る

インストール後に ISO を取り外し忘れているケースが多いです。UTM の「編集 → ドライブ」で ISO を削除してから再起動してみてください。

### 解像度が変わらない

`systemctl status spice-vdagent` でサービスが動いているか確認します。UTM の「動的解像度」が ON になっているかも確認してください。

### 共有フォルダが見えない

`mount | grep 9p` でマウント状況を確認します。fstab の書き方を間違えているとマウントに失敗するので、まず手動マウントが動くか試してみるのがおすすめです。

### 日本語入力が効かない

```bash
# fcitx5 が起動しているか確認
ps aux | grep fcitx5

# 環境変数が設定されているか確認
echo $GTK_IM_MODULE
```

ログアウト → ログインし直すと解決することが多いです。

### Mac との間でコピー＆ペーストが効かない

`spice-vdagent` のインストールを忘れていないか確認してください。

---

## 10. MacのGPUは使えるの？

UTM + ParrotOS を試していて気になったのが「MacのGPUは使えるのか」という点です。結論から言うと、**MacのGPUを VM から直接使うことはできません**。

### なぜ使えないのか

Apple SiliconのGPUはSoC内部に統合されており、PCIeパススルーのような形での仮想マシンへの割り当てに対応していません。Appleがその仕組みを提供していない、という制限です。Intel Macでも同様で、dGPUがあっても仮想マシンへのパススルーはできません。

### 「ハードウェアOpenGL加速」なら使える

UTMの設定で **「ハードウェアOpenGL加速」（virtio-gpu-gl）** を有効にすると、VirGLという仕組みを通じてゲストOSのOpenGLコマンドをホストGPUで処理させることができます。デスクトップのGUI描画が滑らかになる程度の効果はあります。

UTM の「編集 → ディスプレイ」→ **「ハードウェア OpenGL 加速」をオン** にするだけで有効になります。

### 使えること・使えないこと

| 機能 | 使えるか |
|---|---|
| デスクトップのGUI描画を滑らかにする | ✅（VirGL経由） |
| OpenGLアプリの動作改善 | ✅（制限あり） |
| GPU計算（hashcat等のパスワードクラック） | ❌ |
| Metal / CUDA | ❌ |
| 機械学習のGPU高速化 | ❌ |

### セキュリティ用途で困るポイント

最も影響が大きいのは **hashcat**（GPU対応パスワードクラッカー）です。UTM上のParrotOSではCPUモードでしか動かせないため、速度が大幅に落ちます。

```bash
# CPUモードで動かす場合（遅い）
hashcat -m 0 hash.txt wordlist.txt -D 1
```

GPU加速が必要な場合の現実的な代替手段:

- **Mac本体のmacOSで直接実行**: `brew install hashcat` してmacOS上で動かす（Metal経由でGPUが使える）
- **クラウド環境**: AWS p3インスタンス・Google Colab 等でGPU付き環境を使う

---

## 11. その他の制約まとめ

GPU以外にも、UTM上でParrotOSを使うと困る制約がいくつかあります。セキュリティ用途では特に影響が大きいものを整理しておきます。

### WiFiモニターモードが使えない（セキュリティ用途では最大の制約）

MacのWiFiアダプタを仮想マシン側からモニターモードに切り替えることができません。そのため、以下のような無線LAN関連のツールが使えません。

| ツール | 用途 | 使えるか |
|---|---|---|
| `airodump-ng` | WiFiパケットキャプチャ | ❌ |
| `aireplay-ng` | パケットインジェクション | ❌ |
| `kismet` | ワイヤレス調査 | ❌ |

**対策:** モニターモード対応の外付けUSB WiFiアダプタを使う方法がありますが、後述のUSBパススルーの不安定さも絡むため完全な解決策にはなりにくいです。無線LAN攻撃の練習は物理的なLinuxマシンか、専用のハードウェア環境で行う方が確実です。

### USBパススルーが不安定

UTMはUSBデバイスをVMに渡す（パススルー）機能を持っていますが、**Apple Siliconでは動作が不安定**なことがあります。

影響を受けやすいデバイス:
- USB WiFiアダプタ（前述のモニターモード代替手段に影響）
- USBシリアルアダプタ（Flipper Zero等との連携）
- 一部のUSBセキュリティキー・Yubikeyなど

### Bluetoothが使えない

MacのBluetoothアダプタも仮想マシンへのパススルーには対応していません。BLEスキャンやBluetooth関連のセキュリティツールはVM内では動きません。

### スナップショットはQEMUバックエンド限定

UTMの仮想化バックエンドには2種類あります。

| バックエンド | 速度 | スナップショット |
|---|---|---|
| Apple Virtualization | 速い | ❌ 使えない |
| QEMU | やや遅い | ✅ 使える |

実験環境として「壊したら戻す」使い方をしたい場合は、**QEMUバックエンドを選ぶ**必要があります。VM作成時の「仮想化」選択画面で「QEMU」を選べば切り替えられます。

### ネスト仮想化ができない

ParrotOS（VM）の中でさらに仮想マシンを動かす**ネスト仮想化**は非対応です。VM内でDockerを動かすことは問題ありませんが、VM内でKVMやVirtualBoxを使うことはできません。

### パフォーマンスのオーバーヘッド

| 操作 | 影響 |
|---|---|
| CPU集中型ツール（John the Ripper等） | ネイティブより遅い |
| ディスクI/O（ログ解析・forensics） | 仮想ディスク経由のため遅い |
| x86版バイナリのエミュレーション | ARM上のx86エミュは特に遅い |

x86版のバイナリしかないツールをApple Silicon Mac上のUTMで動かすと、さらに一段遅くなります。ARM64版が提供されているツールを優先して使うのが無難です。

### 制約の一覧

| 制約 | 深刻度 | 代替手段 |
|---|---|---|
| GPUパスthrough不可（hashcat等） | 高 | macOS上で直接実行 / クラウド |
| WiFiモニターモード不可 | 高 | 物理Linuxマシン |
| USBパススルーの不安定さ | 中 | macOS側で直接使う |
| Bluetooth不可 | 中 | macOS側で直接使う |
| スナップショット（Apple Virt.不可） | 低 | QEMUバックエンドに切り替え |
| ネスト仮想化不可 | 低 | Dockerで代用できる場合が多い |
| パフォーマンスオーバーヘッド | 低〜中 | ツールによる |

---

## まとめ

UTM + ParrotOS の組み合わせは、想像以上に簡単に動きました。特に Apple Silicon Mac で ARM64 版を使うと動作がサクサクで快適です。

最初はデフォルト設定だと日本語入力や画面調整が使えないので少し戸惑いますが、この記事の手順をやれば問題なく使えるようになります。

ただし、セキュリティ用途では以下の制約を頭に入れておく必要があります。

- **GPUが使えない** → hashcatはmacOS上で実行
- **WiFiモニターモードが使えない** → 無線LAN攻撃の練習は物理環境が必要
- **USBパススルーは不安定** → 外付けデバイス依存のツールは要注意

これらの制約を踏まえたうえで、ネットワーク系・Web系の学習やCTFのフラグ回収など、多くの場面では十分活用できます。

---

## 参考リンク

- [UTM 公式サイト](https://mac.getutm.app)
- [ParrotOS 公式サイト](https://parrotsec.org)
- [ParrotOS ドキュメント](https://parrotsec.org/docs/)
