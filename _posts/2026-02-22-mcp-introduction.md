---
layout: post
title: "MCP（Model Context Protocol）入門：Claude CodeやCursorで使えるAIツール拡張の仕組み"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["mcp", "claudecode", "cursor", "ai", "claude"]
date: "2026-02-22 12:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-02-22-mcp-introduction)でも公開しています。

Claude Codeを使い始めてから、設定ファイルに `mcpServers` という項目があることに気づいていました。「MCP」という言葉もよく見かけるけど、ちゃんと理解できていなかったので、これを機にしっかり調べて自分でも使ってみることにしました。

この記事は「MCPとは何か」を自分なりに整理しつつ、**実際に手を動かして設定するまでの記録**です。同じようにMCPが気になっている方の参考になれば。

## MCPとは

MCP（Model Context Protocol）は、Anthropicが2024年11月に発表したオープンプロトコルです。AIモデルと外部のデータソース・ツール群を接続するための標準的なインターフェースを定義しています。

調べるまで「またAnthropic独自の規格か」と思っていたんですが、GitHubやGoogle、OpenAIなど主要プレイヤーが対応を表明しており、業界標準になりつつある雰囲気があります。

> [公式ドキュメント](https://modelcontextprotocol.io/)

### MCPが解決する問題

「Claude CodeにGitHubを見させたい」「SlackのメッセージをAIに要約させたい」といったことをやろうとすると、従来は各ツールごとに独自の実装が必要でした。

MCPはそこを標準化します：

```
Before MCP:
  Claude ←(独自連携)→ GitHub
  Cursor ←(独自連携)→ GitHub
  → 各ツールごとにGitHub連携を実装

After MCP:
  Claude  ─┐
  Cursor  ─┤←→ MCPサーバー(GitHub) ←→ GitHub API
  VSCode  ─┘
  → MCPサーバーを1つ作れば全ツールで使える
```

### MCPの構成要素

| コンポーネント | 役割 |
|---|---|
| **MCPクライアント** | Claude Code、Cursor などのAIツール |
| **MCPサーバー** | ツールやデータへのアクセスを提供するプログラム |
| **トランスポート** | クライアントとサーバーの通信方式（stdio / HTTP+SSE） |

MCPサーバーが提供できるものは主に3種類あります：

- **Tools（ツール）**: AIが実行できる関数（ファイル操作、API呼び出しなど）
- **Resources（リソース）**: AIが参照できるデータ（ドキュメント、DBの内容など）
- **Prompts（プロンプト）**: 再利用可能なプロンプトテンプレート

---
layout: post
---

## よく使われるMCPサーバー

どんなサーバーがあるか調べてみると、公式とコミュニティ製で結構な数がありました。

### 公式提供（@modelcontextprotocol）

| サーバー | 機能 | パッケージ |
|---|---|---|
| GitHub | リポジトリ・Issue・PR操作 | `@modelcontextprotocol/server-github` |
| Slack | メッセージ送受信・検索 | `@modelcontextprotocol/server-slack` |
| Filesystem | ローカルファイル操作 | `@modelcontextprotocol/server-filesystem` |
| PostgreSQL | DBクエリ実行 | `@modelcontextprotocol/server-postgres` |
| Brave Search | Web検索 | `@modelcontextprotocol/server-brave-search` |
| Fetch | Webページ取得 | `@modelcontextprotocol/server-fetch` |

### コミュニティ製（人気のもの）

- **`@upstash/mcp-server-redis`** — Redisの操作
- **`mcp-server-sqlite`** — SQLiteのクエリ
- **Figma MCP** — Figmaのデザインデータ参照
- **Notion MCP** — Notionページの読み書き

公式の[MCPサーバーリポジトリ](https://github.com/modelcontextprotocol/servers)やコミュニティの[mcp.so](https://mcp.so)でさらに多くのサーバーを探せます。

自分はまずSlackとGitHubのMCPを試してみようと思っています。業務でよく使うツールをAIから直接操作できるようになると、かなり効率が上がりそうです。

---
layout: post
---

## MCPサーバーを自作する

既存のサーバーで足りない場合は自作もできます。Node.js（TypeScript）またはPythonで書けるので、普段使っている言語で試せそうです。

自分はまだここまでは手を出せていませんが、社内ツールと連携させるときに使えると思っているのでメモとして残しておきます。

### シンプルなNode.jsサーバーの例

以下は「現在の天気を返す」シンプルなMCPサーバーの例です。

```bash
npm init -y
npm install @modelcontextprotocol/sdk
```

```typescript
// server.ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

const server = new McpServer({
  name: "weather-server",
  version: "1.0.0",
});

// ツールを定義
server.tool(
  "get_weather",
  "指定した都市の天気情報を取得します",
  {
    city: z.string().describe("都市名（例：Tokyo）"),
  },
  async ({ city }) => {
    // 実際にはAPIを呼び出す
    return {
      content: [
        {
          type: "text",
          text: `${city}の天気: 晴れ、気温 15°C`,
        },
      ],
    };
  }
);

// サーバー起動
const transport = new StdioServerTransport();
await server.connect(transport);
```

```bash
# ビルド
npx tsc

# Claude Codeに登録
claude mcp add weather node /path/to/dist/server.js
```

これだけで、Claude Codeが `get_weather` ツールを使えるようになります。

### Pythonで作る場合

```python
# server.py
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather-server")

@mcp.tool()
def get_weather(city: str) -> str:
    """指定した都市の天気情報を取得します"""
    # 実際にはAPIを呼び出す
    return f"{city}の天気: 晴れ、気温 15°C"

if __name__ == "__main__":
    mcp.run()
```

```bash
pip install mcp
claude mcp add weather python /path/to/server.py
```

---
layout: post
---

## セキュリティの考慮点

MCPサーバーはAIに強力な権限を与えます。利用時は以下の点に注意してください：

1. **信頼できるサーバーのみ追加する** — 不明なコミュニティサーバーは中身を確認してから使用
2. **最小権限の原則** — Filesystemサーバーのアクセス範囲は必要最小限のディレクトリのみ
3. **APIトークンの管理** — トークンは環境変数で渡し、設定ファイルをgitにコミットしない
4. **プロンプトインジェクション** — 悪意あるWebページやファイルの内容を読み込む際に注意

`.gitignore`に設定ファイルを追加するか、ローカル設定とグローバル設定を使い分けることを推奨します。

---
layout: post
---

## 参考リンク

- [Model Context Protocol 公式ドキュメント](https://modelcontextprotocol.io/)
- [MCP公式サーバーリポジトリ（GitHub）](https://github.com/modelcontextprotocol/servers)
- [Claude Code MCPドキュメント](https://docs.anthropic.com/ja/docs/claude-code/mcp)
