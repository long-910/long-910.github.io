---
layout: post
title: "外出中でも GitHub Copilot でコード修正まで完結させる"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["githubcopilot", "github", "mobile", "開発効率化", "ai"]
date: "2026-04-05 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-04-05-github-copilot-mobile)でも公開しています。


## はじめに

「外出中にバグ報告が来た」――こういうとき、以前は「帰宅してから対応する」しかありませんでした。

でも今は違います。**Copilot にコードを修正してもらい、スマホやタブレットから直接コミット・プッシュまで完結できます。**

この記事では、外出先でも開発を前に進められる具体的な方法を紹介します。

---

## 全体像：外出先での修正フロー

```
① バグ報告を受け取る（Slack・メール・GitHub Issue）
       ↓
② GitHub Mobile で該当コードを確認
       ↓
③ Copilot Chat に「修正コードを書いて」と頼む
       ↓
④ github.dev（ブラウザエディタ）に修正コードを貼り付け
       ↓
⑤ コミット・プッシュ → PR 作成まで完了
```

スマホでも「調査して終わり」ではなく、**修正コードの生成からマージまで**持っていけます。

---

## ステップ①：GitHub Mobile でコードを確認する

GitHub の公式アプリ（iOS / Android）で、リポジトリのコードや Issue を確認します。

**インストール:**
- iOS: App Store で「GitHub」を検索
- Android: Google Play Store で「GitHub」を検索

アプリ下部の「Copilot」タブから Copilot Chat が使えます。ここでコードを貼り付けて会話できます。

---

## ステップ②：Copilot Chat で修正コードを生成する

Copilot Chat の真価は「修正案を出す」だけでなく、**そのまま使えるコードを生成してくれる**点にあります。

### バグ修正をお願いする

```
# Slack で届いたエラーと該当コードを貼り付けて

次のエラーが発生しています：
TypeError: Cannot read properties of undefined (reading 'id')
  at processOrder (orders.js:42)

該当コードはこちらです：
function processOrder(order) {
  const userId = order.user.id;  // ← ここで落ちている
  return createInvoice(userId, order.items);
}

修正したコードを書いてください。
```

Copilot は修正済みのコード全体を返してくれます：

```javascript
// Copilot が生成した修正コード
function processOrder(order) {
  if (!order?.user?.id) {
    throw new Error(`Invalid order: missing user.id (orderId: ${order?.id})`);
  }
  const userId = order.user.id;
  return createInvoice(userId, order.items);
}
```

### リファクタリングをお願いする

```
このコードを async/await に書き直してください。
動作は変えずに、エラーハンドリングも適切に入れてください。

[コードを貼り付け]
```

### テストコードを書いてもらう

```
このコードに対して Jest のユニットテストを書いてください。
正常系・異常系を網羅してください。

[コードを貼り付け]
```

---

## ステップ③：github.dev でコードを編集・コミットする

Copilot が生成したコードを実際にリポジトリに反映させます。PC 不要でブラウザだけで完結します。

### github.dev の開き方

スマホ・タブレットのブラウザで GitHub リポジトリを開き、URL の `github.com` を **`github.dev`** に書き換えるだけです。

```
変更前: https://github.com/your-org/your-repo
変更後: https://github.dev/your-org/your-repo
```

VS Code そのものがブラウザで起動します。

### 修正コードを貼り付けてコミットする

1. 左のファイルツリーから対象ファイルを開く
2. Copilot が生成したコードを貼り付ける
3. 左のサイドバーの「Source Control」アイコンをタップ
4. コミットメッセージを入力して「✓ Commit & Push」

これだけでリモートブランチにプッシュされます。

---

## Codespaces を使えばさらに強力に

Codespaces はブラウザ上でフルのサーバー環境が動きます。**テストの実行やパッケージのインストールも外出先でできます。**

### Codespaces の起動方法

GitHub リポジトリの「Code」ボタン →「Codespaces」→「Create codespace on main」

### Copilot との組み合わせ

Codespaces 内の VS Code には Copilot が統合されています。

```
ターミナルで npm test を実行
→ テストが落ちる
→ Copilot Chat に「このテストエラーを修正してください」と頼む
→ 修正コードを適用してもう一度 npm test
→ パスしたらコミット
```

外出先でも「修正 → テスト → コミット」の開発サイクルが回せます。

---

## 実践例：バグ報告への対応を外出先で完結させる

### シナリオ

移動中に Slack で「本番でエラーが出てる」と連絡が来た。

### 対応フロー

**1. GitHub Mobile でエラーの Issue を確認**

```
Copilot Chat:「この Issue の内容を要約して、
どのファイルが原因か教えてください」
```

**2. 該当ファイルを開いてコードをコピー**

GitHub Mobile のファイルビューアで該当箇所を長押し → コピー

**3. Copilot Chat で修正コードを生成**

```
「このコードで NullPointerException が発生しています。
修正したコード全体を書いてください。

[コードを貼り付け]」
```

**4. github.dev で修正を適用**

ブラウザで `github.dev/...` を開き、生成されたコードを貼り付けてコミット。

**5. PR を作成してレビュー依頼**

Copilot に PR の説明文も書いてもらいます：

```
「この修正内容を PR の説明文として書いてください。
変更点・影響範囲・テスト方法を含めてください」
```

GitHub の PR 作成画面に貼り付けて完了。

---

## iPad + 外付けキーボードが最強の外出先環境

スマホだけでも上記の作業はできますが、iPad + 外付けキーボードがあると快適さが大きく変わります。

| 環境 | できること |
|------|-----------|
| スマホのみ | Copilot でコード生成・github.dev で小規模な修正 |
| iPad（タッチのみ） | 上記 + Codespaces でテスト実行 |
| iPad + キーボード | ほぼ PC と同等の開発体験 |

Codespaces + iPad + キーボードの組み合わせは、カフェやコワーキングスペースで本格的な開発ができる環境です。

---

## Copilot に頼む際のコツ

外出先は PC より操作が制約されるため、**一度のやり取りで完結するプロンプト**を意識すると効率的です。

```
# 良い頼み方の例

「次のコードを修正してください。
条件：
- 動作は変えない
- TypeScript の型エラーを解消する
- 関数全体を出力してください（部分的なコードは不要）

[コードを貼り付け]」
```

ポイントは「**関数全体を出力してください**」と明示することです。部分的なコードが返ってくると、スマホ画面での確認・貼り付けが面倒になります。

---

## まとめ

| 方法 | 主な用途 |
|------|---------|
| GitHub Mobile + Copilot Chat | コード生成・バグ調査・Issue 対応 |
| github.dev（ブラウザ） | 修正コードの貼り付け・コミット・PR 作成 |
| GitHub Codespaces | テスト実行・複雑な修正 |

「外出中は帰宅待ち」ではなく、**Copilot にコードを書いてもらって外出先から修正を完了できる**時代になっています。移動時間や隙間時間を使って、バグ修正・ドキュメント更新・PR レビューを前に進めてみてください。

## 参考

- [GitHub Mobile 公式サイト](https://github.com/mobile)
- [github.dev Web Editor ドキュメント](https://docs.github.com/ja/codespaces/the-githubdev-web-based-editor)
- [GitHub Codespaces ドキュメント](https://docs.github.com/ja/codespaces)
- [GitHub Copilot Chat ドキュメント](https://docs.github.com/ja/copilot/github-copilot-chat)
