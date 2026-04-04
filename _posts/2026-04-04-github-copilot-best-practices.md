---
layout: post
title: "GitHub Copilot 実践活用術 ― 日常開発を加速する10のテクニック"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["githubcopilot", "ai", "vscode", "開発効率化", "ペアプログラミング"]
date: "2026-04-04 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-04-04-github-copilot-best-practices)でも公開しています。


## はじめに

GitHub Copilot を導入してから、コードを書くスピードが変わりました。しかし「補完が出るから速い」という話ではなく、**Copilot をどう使うか**によって体験の質が大きく変わることに気づきました。

この記事では、実際の開発の中で試してきた活用テクニックをまとめます。IDE での使い方から Copilot CLI、さらには GitHub の各機能との連携まで幅広くカバーします。

---

## 1. コメントを「仕様書」として書く

Copilot の補完精度は、**コンテキストの質**に直結します。

```python
# ユーザーID と期間（start_date, end_date）を受け取り、
# その期間内に購入したアイテムの合計金額を返す。
# 購入データは orders テーブルにある。user_id が一致するレコードを集計すること。
def get_total_purchase(user_id: int, start_date: date, end_date: date) -> Decimal:
```

このように**何をするか・どのデータを使うか**をコメントに書くと、Copilot が的確な実装を生成してくれます。「関数名だけ書いて待つ」より、明確な意図を伝えるほうが精度が上がります。

---

## 2. `# Example:` でサンプルを示す

入出力の例を書くと、型や形式のミスが減ります。

```typescript
// メールアドレスのドメイン部分を抽出する
// Example:
//   extractDomain("user@example.com") => "example.com"
//   extractDomain("admin@sub.domain.org") => "sub.domain.org"
function extractDomain(email: string): string {
```

テスト駆動開発の感覚でコメントを書くと、Copilot の生成コードが想定通りになりやすいです。

---

## 3. 既存コードの近くに新しいコードを書く

Copilot はファイル内の**既存コードをコンテキスト**として参照します。

```go
// 既存の実装
func (s *UserService) GetByID(ctx context.Context, id int) (*User, error) {
    return s.repo.FindByID(ctx, id)
}

// ここに新しい関数を書くと、上の関数のスタイルを踏襲した補完が出る
func (s *UserService) GetByEmail(ctx context.Context, email string) (*User, error) {
```

新しい関数を書く際は、似た既存関数の**直下または近く**に配置すると、命名規則・エラーハンドリングのスタイルを学習した補完が得られます。

---

## 4. Copilot Chat でコードレビューを頼む

IDE のチャット機能は、補完だけでなくコードの説明・改善提案にも使えます。

```
/explain このコードの時間計算量と、改善できる点があれば教えてください
```

```
/fix このコードのバグを修正してください
```

```
/tests この関数のユニットテストを生成してください
```

スラッシュコマンドを使うと意図が明確になり、的確な回答が返ってきやすくなります。

---

## 5. GitHub Copilot CLI でターミナルを強化する

`gh extension install github/gh-copilot` でインストールできる CLI ツールです。

```bash
# コマンドの意味を説明してもらう
gh copilot explain "git rebase -i HEAD~3"

# やりたいことを自然言語で伝えてコマンドを生成
gh copilot suggest "1週間以内に更新されたファイルを検索したい"
```

シェルの alias として設定すると使いやすくなります：

```bash
# .zshrc または .bashrc
alias '??'='gh copilot explain'
alias '!'='gh copilot suggest'
```

複雑な `find` や `awk`、`jq` コマンドを毎回調べる手間が省けます。

---

## 6. Pull Request の説明文を自動生成する

GitHub の PR 作成画面で **Copilot ボタン**（✨ アイコン）を押すと、コミット差分から PR の説明文を自動生成してくれます。

自動生成される内容：

- 変更の概要
- 実装した内容の箇条書き
- テスト内容

生成された文章を**そのまま使うのではなく、編集のベースとして使う**のがポイントです。特に「なぜこの実装にしたか」という背景は手動で追記すると PR の質が上がります。

---

## 7. `.github/copilot-instructions.md` でプロジェクト固有のルールを定義する

リポジトリに `copilot-instructions.md` を置くことで、Copilot Chat がプロジェクト固有のコンテキストを参照するようになります。

```markdown
# Copilot Instructions

## プロジェクト概要
このリポジトリは E コマースサイトのバックエンド API です。

## 技術スタック
- Python 3.12 / FastAPI
- PostgreSQL 16
- 認証: JWT (jose ライブラリ使用)

## コーディングルール
- エラーは例外ではなく Result 型で返す（`returns` ライブラリ使用）
- DB アクセスは必ず `async/await` を使う
- 型アノテーションは必須

## テスト
- pytest を使用
- テストファイルは `tests/` ディレクトリに置く
- モックは `pytest-mock` を使用
```

これにより、Copilot Chat への質問がプロジェクトの文脈で解釈されるようになります。

---

## 8. コードブロックを選択して質問する

VS Code では、コードを**選択した状態で Copilot Chat を開く**と、そのコードを対象に質問できます。

実用的なユースケース：

```
# 選択したコードに対して
「このコードをより Pythonic に書き直してください」
「SQL インジェクションのリスクはありますか」
「このロジックを図解してください」
```

ファイル全体を貼り付けるより、**問題のある箇所だけ選択して聞く**ほうがノイズが少なく、的確な回答が得られます。

---

## 9. Copilot を使った正規表現・クエリの生成

正規表現や SQL クエリは記述ミスが起きやすい領域です。自然言語で説明して生成させると効率が上がります。

```
# Copilot Chat への質問例
「日本の郵便番号（xxx-xxxx 形式）にマッチする正規表現を作ってください。
 入力は 7 桁の数字で、4 桁目にハイフンが入ることもあります」
```

```sql
-- Copilot に書かせる SQL の例
-- 2025年に登録したユーザーのうち、
-- 過去30日間に1回もログインしていないユーザーの ID と email を取得
```

生成されたコードは必ず動作確認・レビューをしてください。正規表現は特にエッジケースの確認が重要です。

---

## 10. 繰り返しパターンは Copilot に「学習」させる

同じスタイルの実装を複数書く際、最初の 2〜3 件を丁寧に実装しておくと、Copilot が残りを推測して補完してくれます。

```typescript
// 1つ目（丁寧に書く）
export const getUser = createAsyncThunk(
  'user/getUser',
  async (id: number, { rejectWithValue }) => {
    try {
      const response = await api.get(`/users/${id}`);
      return response.data;
    } catch (error) {
      return rejectWithValue((error as Error).message);
    }
  }
);

// 2つ目（途中まで書くと残りが補完される）
export const updateUser = createAsyncThunk(
  'user/updateUser',
  // ← ここで Tab を押すと上と同じパターンが補完される
```

Redux Toolkit の thunk、React のカスタムフック、API クライアントの各エンドポイントなど、**定型パターンが多い実装**で特に効果的です。

---

## まとめ

GitHub Copilot の効果を最大化するポイントをまとめます。

| テクニック | 効果 |
|----------|------|
| コメントを仕様書として書く | 補完精度が向上 |
| サンプル入出力を示す | 型・形式のミスが減る |
| 既存コードの近くに書く | スタイルが統一される |
| Chat でコードレビュー | バグ発見・品質向上 |
| CLI でターミナル強化 | コマンド調査の手間が省ける |
| PR 説明文の自動生成 | ドキュメント作成の効率化 |
| copilot-instructions.md | プロジェクト固有のコンテキスト |
| 選択してから質問 | 的確な回答が得られる |
| 正規表現・クエリを生成 | 記述ミスの削減 |
| パターンを先に実装する | 繰り返し実装の高速化 |

GitHub Copilot は「ただの補完ツール」ではなく、**使い方次第でペアプログラマー並みの存在**になります。ぜひ日々の開発の中で試してみてください。

## 参考

- [GitHub Copilot 公式ドキュメント](https://docs.github.com/ja/copilot)
- [GitHub Copilot CLI](https://docs.github.com/ja/copilot/github-copilot-in-the-cli)
- [Copilot Instructions ファイルの設定](https://docs.github.com/ja/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
