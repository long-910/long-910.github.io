---
layout: post
title: "YouCam APIが気になったので調べてみた - ARと生成AIで美容体験を作るとは"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["youcam", "ar", "api", "ai", "画像処理"]
date: "2026-05-30 10:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-30-youcam-api-investigation)でも公開しています。


## きっかけ

ZennFesのテーマ一覧を眺めていたら「YouCam APIを活用した実装事例とアイデア」という項目が目に入りました。YouCamといえばメイクアプリのイメージがあったのですが、**開発者向けのAPI**があることを初めて知りました。

美容系のAR/AIをAPIとして呼び出せるなら、面白いアプリが作れそうです。実際に触ってみる前に、まずどんなことができるのかを調べてみました。

---

## YouCam APIとは

YouCam APIは、台湾の企業 [Perfect Corp.](https://www.perfectcorp.com) が提供する**美容・ファッション特化のAI/AR機能群をREST APIで利用できるサービス**です。

同社は「YouCam Makeup」という世界累計3億ダウンロードを超えるメイクアプリを開発しており、そこで培ったAR技術をAPIとして外部開発者に開放しています。

### 主な機能カテゴリ

| カテゴリ | 機能例 |
|---|---|
| 肌分析 | 毛穴・しわ・シミ・くすみのスコアリング |
| メイク試着 | リップ・チーク・アイシャドウのARバーチャル試着 |
| ヘアカラー | 100色以上のヘアカラーバーチャル試着 |
| ファッション | 服・アクセサリーのバーチャル試着 |
| 顔解析 | 顔型・顔属性（年齢・表情など）の推定 |
| 画像生成 | AIによるポートレート生成・背景変換 |

これだけの機能がまとまっているAPIは珍しいと感じました。美容・ファッション領域に特化しているからこそ、汎用の画像認識APIより精度が高い部分があるようです。

---

## APIの始め方

### アカウント作成とAPIキー取得

1. [YouCam Online Editor](https://yce.perfectcorp.com) にアクセス
2. 無料アカウントを作成
3. API Consoleからキーを発行

無料枠でも一定のクレジットがもらえるので、試作には十分そうです。

### 料金体系の概要

処理はクレジット消費型で、おおよそ以下のイメージです。

- 画像1枚の処理：1〜2クレジット
- 動画：1クレジットで約2秒分処理

詳細は[料金ページ](https://yce.perfectcorp.com/ja/ai-api/api-pricing)で確認できます。

---

## API V2 の基本的な使い方

2024年にV2がリリースされ、レスポンス形式がシンプルになりました。以前は結果がZIPで返ってきていたのが、V2では**直接JSONに画像URLやスコアが含まれる**形式になっています。

基本的な流れはこうです。

```
1. 画像をアップロード → task_id を取得
2. task_id でポーリング（または webhook）→ 処理完了を待つ
3. 結果の画像URL / スコアを受け取る
```

### 肌分析APIの呼び出しイメージ（Python）

```python
import requests
import time

API_KEY = "your_api_key_here"
BASE_URL = "https://api.perfectcorp.com/v2"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json",
}

# 画像URLを渡してタスクを作成
payload = {
    "image_url": "https://example.com/face.jpg",
    "concerns": ["pores", "wrinkles", "dark_spots"],  # 分析したい肌の悩み
}

res = requests.post(f"{BASE_URL}/skin-analysis", json=payload, headers=headers)
task_id = res.json()["task_id"]

# ポーリングで結果を取得
while True:
    result = requests.get(f"{BASE_URL}/tasks/{task_id}", headers=headers).json()
    if result["status"] == "completed":
        print(result["scores"])  # 各項目のスコアが返る
        break
    time.sleep(2)
```

実際のエンドポイント名やパラメータは[公式ドキュメント](https://yce.perfectcorp.com/document/index.html)で確認が必要ですが、構造はこういったイメージです。

### 注意点として分かったこと

調べる中で、Zennの他の記事で実際に試している方の知見が参考になりました。

- **SDモードとHDモードは混在不可**：肌分析のリクエスト内でSD/HDの診断項目を混ぜるとエラーになる
- **事前バリデーションを入れると良い**：アップロード直後に「顔が検出されているか」「解像度・明るさは問題ないか」を確認してからAPI呼び出しに進む設計が安定する
- **非同期前提**：処理に数秒かかるため、ポーリングまたはWebhookの実装が必要

---

## 最近の動き：MCPサポートと「Ask AI」

2026年5月、Perfect Corp.はYouCam APIプラットフォームに**「Ask AI」**という無料のAIアシスタントを統合しました。

これは、MCP（Model Context Protocol）対応クライアント（CursorやVS CodeのCopilot、Claude Desktopなど）の設定ファイルにAPIキーを1行追加するだけで、自然言語でYouCam APIの機能を呼び出せるようにするものです。

```json
// .cursor/mcp.json などに追加するだけ
{
  "mcpServers": {
    "youcam": {
      "command": "npx",
      "args": ["@perfectcorp/youcam-mcp"],
      "env": {
        "YOUCAM_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

英語ドキュメントへの敷居が下がるのと、「この画像の肌分析して」と日本語で指示するだけで動く点は、試し始めのハードルがかなり低くなるように感じました。

---

## 実装してみたいアイデア

調べていて「これ作れそう」と思ったアイデアをいくつかメモしました。

### 1. スキンケア記録アプリ

毎日の肌写真を撮って送ると、毛穴・シミ・くすみのスコアを記録・グラフ化してくれるアプリ。「最近の睡眠不足が肌に出てるな」みたいな振り返りができると面白い。

```
定期撮影 → 肌分析API → スコアをDB保存 → 折れ線グラフで可視化
```

### 2. メイクレシピ共有サービス

メイクの仕上がり画像とメイク設定（口紅の色、アイシャドウの種類）をセットで投稿・共有できるサービス。YouCam APIでそのメイクを別の顔に再現できれば「このメイク、自分に合うかな？」を確認しながら探せる。

```
投稿されたメイクパラメータ → 自分の顔写真に適用 → バーチャル試着
```

### 3. ヘアカラーシミュレーター

「この服に合うヘアカラーは？」という問いに答えるツール。服装の画像から色を抽出して、相性の良いヘアカラーをYouCam APIで試着させる。

```
服の色抽出（カラーパレット） → ヘアカラーAPI で複数色を試着 → 比較表示
```

### 4. EC サイトの「顔に合う商品」レコメンド

化粧品ECに組み込んで、顔型・肌色診断の結果をもとに「あなたの顔型にはこのリップ形状が似合います」というレコメンドを出す。

---

## 感想

調べてみると、**美容・ファッション特化のAI APIという珍しいポジション**を持ったサービスだと分かりました。汎用の画像認識AIで同じことをやろうとすると、顔ランドマーク検出・メイクのマスク生成・色合成など多くのパーツを自分で組み合わせる必要があります。それをAPIひとつで解決できるのは開発コストの面でかなり助かりそうです。

一方で、クレジット消費型の課金なのでトラフィックが読めないサービスでのコスト試算は事前にしっかりやっておく必要があります。

次は実際にAPIキーを取得して、肌分析を動かすところまでやってみたいと思います。

---

## 参考リンク

- [YouCam API 公式ドキュメント](https://yce.perfectcorp.com/document/index.html)
- [YouCam API 料金ページ](https://yce.perfectcorp.com/ja/ai-api/api-pricing)
- [Perfect Corp. 「Ask AI」統合のプレスリリース](https://www.businesswire.com/news/home/20260513560473/en/Perfect-Corp.-Integrates-Free-AI-Assistant-Ask-AI-into-YouCam-API-Platform)
- [YouCam API V2 の動作検証（Zenn）](https://zenn.dev/nakamura1962/articles/youcam-ukiyoe-survey)
