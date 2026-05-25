---
layout: post
title: "AmiVoice API × Claude APIで作るリアルタイム日本語音声対話システム"
img_path: /assets/img/logos
image:
  path: logo-only.svg
  width: 100%
  height: 100%
  alt: Zenn
category: [Tech]
tags: ["amivoice", "claude", "音声認識", "生成AI", "nodejs"]
date: "2026-05-25 09:00"
---


---

この記事は[Zenn](https://zenn.dev/long910/articles/2026-05-25-amivoice-claude-voice-assistant)でも公開しています。


## はじめに

「日本語の音声認識を使ったAIアシスタントを作りたい」と思ったとき、まず壁になるのが**認識精度**です。汎用エンジンでは専門用語・固有名詞・日本語特有の表現が崩れやすく、生成AIへの入力品質が落ちてしまいます。

本記事では、日本語特化の音声認識API「**AmiVoice API**」と「**Claude API**」を組み合わせ、WebSocketストリーミングによるリアルタイム音声対話システムを構築します。公式ドキュメントをもとにプロトコル仕様を正確に押さえながら実装します。

### 完成するシステムの概要

- ブラウザのマイクから音声を取得し、PCM形式でサーバーへ送信
- AmiVoice APIのWebSocketで**リアルタイムストリーミング認識**
- 認識確定テキストをClaude APIに渡して**自然な応答をストリーミング生成**

---

## AmiVoice APIとは

[AmiVoice API](https://acp.amivoice.com/amivoice_api/)はアドバンスト・メディアが提供する日本語特化の音声認識クラウドサービスです。汎用エンジンに加え、医療・金融・保険など専門領域向けのモデルも揃っており、特殊用語を多用する業務システムでの採用実績があります。

### 料金体系

| エンジン | ログあり | ログなし |
|----------|----------|----------|
| 汎用（-a-general） | 99円/時間 | 158円/時間 |
| 医療・電子カルテ | 297円/時間 | 475円/時間 |

**毎月60分まで全エンジン無料**で利用できます。開発・検証フェーズはコストゼロでスタートできます。

### 3種類のAPIインターフェース

AmiVoice APIには用途に応じた3つのインターフェースがあります。

| インターフェース | 特徴 | 向いているシーン |
|----------------|------|----------------|
| **WebSocket** | 双方向ストリーミング、リアルタイム認識 | 音声対話、会議リアルタイム字幕 |
| **同期HTTP** | 音声ファイルをPOST→即レスポンス（16MB以下） | 短い録音ファイルの文字起こし |
| **非同期HTTP** | ジョブキュー方式、大容量ファイル対応、感情解析オプションあり | 長時間音声のバッチ処理、コールセンター録音 |

リアルタイム対話には**WebSocket**一択です。本記事もこのインターフェースを使います。

---

## システム構成

```
[ブラウザ]
  │  Web Audio API でマイク音声取得（PCM 16kHz / LE）
  │  WebSocket でサーバーへ送信
  ▼
[Node.js サーバー]
  │  ブラウザ ↔ AmiVoice を中継（APIキーをサーバー側に隠蔽）
  │  A イベント（認識確定）受信 → Claude API 呼び出し
  │  SSE でクライアントへストリーミング返答
  ▼
[AmiVoice API]     [Claude API]
  音声認識           応答生成
```

APIキーをブラウザに渡さないため、**Node.jsをプロキシサーバーとして使う**構成にします。

---

## AmiVoice WebSocket プロトコルの仕様

実装前にプロトコルを正確に理解しておきます。

### 接続エンドポイント

```
wss://acp-api.amivoice.com/v1/        # ログあり（デフォルト）
wss://acp-api.amivoice.com/v1/nolog/  # ログなし
```

### コマンドフロー

```
クライアント → サーバー
─────────────────────────────────────────────
① s コマンド（テキストフレーム）
  s <audioFormat> <grammarFileNames> authorization=<APPKEY>

  例: s LSB16K -a-general authorization=YOUR_APP_KEY

  ※ audioFormat は送信PCMの形式に合わせる
     LSB16K = 16kHz リトルエンディアン（ブラウザのWebAudio APIはLE）
     MSB16K = 16kHz ビッグエンディアン

② 音声データ（バイナリフレーム）
  先頭バイトを 0x70（'p' のASCIIコード）にし、
  続けてPCM音声データをそのまま連結する

③ e コマンド（テキストフレーム）
  音声送信完了を通知
  
サーバー → クライアント（イベントパケット）
─────────────────────────────────────────────
s   sコマンド成功（この応答が来たら音声データ送信を開始できる）
S   音声区間開始（VADが発話を検出）
U   中間認識結果（発話中に逐次更新される JSON）
A   最終認識結果（発話区間の確定テキスト JSON）← ここでClaude APIを呼ぶ
E   音声区間終了
e   eコマンド完了
```

### A/Uイベントのレスポンス形式

AイベントとUイベントのメッセージは `A ` または `U ` に続いてJSONが入ります。

```json
A {
  "text": "今日の東京の天気を教えてください",
  "results": [
    {
      "text": "今日の東京の天気を教えてください",
      "tokens": [
        { "written": "今日", "starttime": 0,   "endtime": 320,  "confidence": 0.98 },
        { "written": "の",   "starttime": 320,  "endtime": 400,  "confidence": 0.99 },
        { "written": "東京", "starttime": 400,  "endtime": 720,  "confidence": 0.97 }
      ],
      "starttime": 0,
      "endtime": 2100
    }
  ],
  "utteranceid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "code": "",
  "message": ""
}
```

**UイベントはAイベントの前に複数回届く中間結果**なので、UIのリアルタイム表示には使いますがAI呼び出しのトリガーにはしません。

---

## 実装

### 環境準備

```bash
mkdir voice-assistant && cd voice-assistant
npm init -y
npm install express ws @anthropic-ai/sdk dotenv
```

```env
# .env
AMIVOICE_APP_KEY=your_amivoice_app_key
ANTHROPIC_API_KEY=your_anthropic_api_key
PORT=3000
```

### サーバー実装（`server.js`）

```javascript
import express from 'express';
import { createServer } from 'http';
import { WebSocketServer, WebSocket } from 'ws';
import Anthropic from '@anthropic-ai/sdk';
import 'dotenv/config';

const app = express();
const server = createServer(app);
const wss = new WebSocketServer({ server, path: '/ws' });
const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// ログあり: wss://acp-api.amivoice.com/v1/
// ログなし: wss://acp-api.amivoice.com/v1/nolog/
const AMIVOICE_WS_URL = 'wss://acp-api.amivoice.com/v1/nolog/';

const sessions = new Map();

wss.on('connection', (clientWs) => {
  const sessionId = crypto.randomUUID();
  const session = { history: [], amiWs: null, amiReady: false };
  sessions.set(sessionId, session);

  // AmiVoice への WebSocket 接続
  const amiWs = new WebSocket(AMIVOICE_WS_URL);
  session.amiWs = amiWs;

  amiWs.on('open', () => {
    // s コマンド送信: s <audioFormat> <grammarFileNames> authorization=<APPKEY>
    // LSB16K = 16kHz リトルエンディアン（Web Audio API の出力形式に合わせる）
    const sCommand = `s LSB16K -a-general authorization=${process.env.AMIVOICE_APP_KEY}`;
    amiWs.send(sCommand);
  });

  amiWs.on('message', async (raw) => {
    const text = raw.toString();

    if (text === 's') {
      // s コマンド成功 → 音声データ受け付け可能になった
      session.amiReady = true;
      clientWs.send(JSON.stringify({ type: 'ready' }));
      return;
    }

    if (text.startsWith('U ')) {
      // 中間認識結果 → UIのリアルタイム表示に使う（AI呼び出しはしない）
      try {
        const json = JSON.parse(text.slice(2));
        if (json.text) {
          clientWs.send(JSON.stringify({ type: 'partial', text: json.text }));
        }
      } catch { /* ignore parse errors */ }
      return;
    }

    if (text.startsWith('A ')) {
      // 最終認識結果 → Claude API を呼び出す
      try {
        const json = JSON.parse(text.slice(2));
        const utterance = json.text?.trim();
        if (!utterance) return;

        clientWs.send(JSON.stringify({ type: 'recognized', text: utterance }));
        await generateResponse(sessionId, utterance, clientWs);
      } catch (err) {
        console.error(`[${sessionId}] Parse error:`, err.message);
      }
      return;
    }

    // S(音声区間開始) / E(音声区間終了) / e(eコマンド完了) は無視
  });

  amiWs.on('error', (err) => {
    console.error(`AmiVoice error [${sessionId}]:`, err.message);
    clientWs.send(JSON.stringify({ type: 'error', message: 'AmiVoice error' }));
  });

  amiWs.on('close', () => {
    session.amiReady = false;
  });

  // ブラウザからのメッセージを処理
  clientWs.on('message', (data) => {
    if (!session.amiReady || amiWs.readyState !== WebSocket.OPEN) return;

    if (Buffer.isBuffer(data)) {
      // 音声PCMデータ: 先頭バイトを 0x70('p') にして AmiVoice へ転送
      const frame = Buffer.concat([Buffer.from([0x70]), data]);
      amiWs.send(frame);
    } else {
      // テキストコマンド: stop のみ受け付ける
      try {
        const msg = JSON.parse(data.toString());
        if (msg.type === 'stop') {
          amiWs.send('e');  // 音声送信終了を通知
        }
      } catch { /* ignore */ }
    }
  });

  clientWs.on('close', () => {
    if (amiWs.readyState === WebSocket.OPEN) amiWs.close();
    sessions.delete(sessionId);
  });
});

async function generateResponse(sessionId, userText, clientWs) {
  const session = sessions.get(sessionId);
  if (!session) return;

  session.history.push({ role: 'user', content: userText });
  if (session.history.length > 20) session.history = session.history.slice(-20);

  clientWs.send(JSON.stringify({ type: 'ai_start' }));

  let fullResponse = '';
  try {
    const stream = await anthropic.messages.stream({
      model: 'claude-sonnet-4-6',
      max_tokens: 1024,
      system: `あなたは親切な音声アシスタントです。
ユーザーの質問に簡潔かつ丁寧な日本語で答えてください。
回答は音声で読み上げることを前提に、箇条書き記号（・-）や
コードブロックは避け、自然な話し言葉で記述してください。`,
      messages: session.history,
    });

    for await (const chunk of stream) {
      if (
        chunk.type === 'content_block_delta' &&
        chunk.delta.type === 'text_delta'
      ) {
        fullResponse += chunk.delta.text;
        clientWs.send(JSON.stringify({ type: 'ai_token', text: chunk.delta.text }));
      }
    }

    session.history.push({ role: 'assistant', content: fullResponse });
    clientWs.send(JSON.stringify({ type: 'ai_end' }));
  } catch (err) {
    console.error(`Claude error [${sessionId}]:`, err.message);
    clientWs.send(JSON.stringify({ type: 'error', message: 'AI error' }));
  }
}

app.use(express.static('public'));
server.listen(process.env.PORT, () => {
  console.log(`http://localhost:${process.env.PORT}`);
});
```

### フロントエンド実装（`public/index.html`）

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>音声対話アシスタント</title>
  <style>
    body { font-family: sans-serif; max-width: 640px; margin: 40px auto; padding: 0 16px; }
    #status { color: #666; font-size: 0.9em; min-height: 1.4em; }
    #transcript { padding: 12px; background: #f5f5f5; border-radius: 8px; min-height: 40px; }
    #response   { padding: 12px; background: #e8f4fd; border-radius: 8px; min-height: 80px; margin-top: 12px; }
    button { padding: 12px 32px; font-size: 1rem; border-radius: 8px; border: none; cursor: pointer; margin-top: 16px; }
    #btn-start { background: #4CAF50; color: white; }
    #btn-stop  { background: #f44336; color: white; display: none; }
    .partial   { color: #aaa; font-style: italic; }
  </style>
</head>
<body>
  <h1>🎙️ 音声対話アシスタント</h1>
  <div id="status">接続中...</div>
  <div id="transcript">（認識テキストが表示されます）</div>
  <div id="response">（AIの返答が表示されます）</div>
  <button id="btn-start" disabled>録音開始</button>
  <button id="btn-stop">録音停止</button>

  <script>
    const ws = new WebSocket(`ws://${location.host}/ws`);
    const statusEl    = document.getElementById('status');
    const transcriptEl = document.getElementById('transcript');
    const responseEl  = document.getElementById('response');
    const btnStart    = document.getElementById('btn-start');
    const btnStop     = document.getElementById('btn-stop');

    let audioCtx, stream, processor;

    ws.onmessage = ({ data }) => {
      const msg = JSON.parse(data);
      switch (msg.type) {
        case 'ready':
          statusEl.textContent = '待機中（録音ボタンを押してください）';
          btnStart.disabled = false;
          break;
        case 'partial':
          transcriptEl.innerHTML = `<span class="partial">${msg.text}…</span>`;
          break;
        case 'recognized':
          transcriptEl.textContent = msg.text;
          responseEl.textContent = '考え中...';
          break;
        case 'ai_start':
          responseEl.textContent = '';
          break;
        case 'ai_token':
          responseEl.textContent += msg.text;
          break;
        case 'ai_end':
          statusEl.textContent = '録音中... 話しかけてください';
          break;
        case 'error':
          statusEl.textContent = `エラー: ${msg.message}`;
          break;
      }
    };

    btnStart.addEventListener('click', async () => {
      stream = await navigator.mediaDevices.getUserMedia({ audio: true });

      // AmiVoice は 16kHz 推奨。AudioContext のサンプリングレートを明示的に指定
      audioCtx = new AudioContext({ sampleRate: 16000 });
      const source = audioCtx.createMediaStreamSource(stream);

      // ScriptProcessorNode で PCM データを取得
      // （モダン環境では AudioWorklet を推奨するが、ここではシンプルさを優先）
      processor = audioCtx.createScriptProcessor(4096, 1, 1);
      processor.onaudioprocess = (e) => {
        const float32 = e.inputBuffer.getChannelData(0);
        // Float32 → Int16 変換（リトルエンディアン）
        const int16 = new Int16Array(float32.length);
        for (let i = 0; i < float32.length; i++) {
          const s = Math.max(-1, Math.min(1, float32[i]));
          int16[i] = s < 0 ? s * 0x8000 : s * 0x7FFF;
        }
        // バイナリデータとして送信（サーバー側で 'p' プレフィックスを付ける）
        ws.send(int16.buffer);
      };
      source.connect(processor);
      processor.connect(audioCtx.destination);

      btnStart.style.display = 'none';
      btnStop.style.display  = 'inline-block';
      statusEl.textContent   = '録音中... 話しかけてください';
    });

    btnStop.addEventListener('click', () => {
      ws.send(JSON.stringify({ type: 'stop' }));
      processor?.disconnect();
      stream?.getTracks().forEach(t => t.stop());
      audioCtx?.close();

      btnStop.style.display  = 'none';
      btnStart.style.display = 'inline-block';
      statusEl.textContent   = '待機中...';
    });
  </script>
</body>
</html>
```

### 動作確認

```bash
node --env-file=.env server.js
# ブラウザで http://localhost:3000 を開く
```

---

## 実装のポイントと注意事項

### 1. s コマンドのフォーマットは厳密

sコマンドはスペース区切りのテキストフレームで、順序が固定されています。

```
s <audioFormat> <grammarFileNames> [key=value ...]
```

よくある間違いとして、フラグ形式（`-a`, `-l`など）を使いたくなりますが、AmiVoice APIはそれに対応していません。認証情報も `authorization=APPKEY` というキーバリュー形式です。

### 2. 音声フォーマットの指定

ブラウザの Web Audio API は x86/x64 環境ではリトルエンディアンのPCMを出力します。そのため audioFormat には `LSB16K` を指定します。`MSB16K`（ビッグエンディアン）を指定すると認識精度が著しく落ちます。

| audioFormat | バイト順 | サンプリングレート |
|-------------|---------|-----------------|
| LSB16K | LE（リトルエンディアン） | 16kHz |
| MSB16K | BE（ビッグエンディアン） | 16kHz |
| LSB8K  | LE | 8kHz |
| MSB8K  | BE | 8kHz |

### 3. 音声データ送信の形式

音声データは**バイナリフレームの先頭バイトを `0x70`（'p' のASCIIコード）**にして送ります。サーバー実装では `Buffer.concat([Buffer.from([0x70]), pcmData])` の形で付加しています。

### 4. A イベントと U イベントの使い分け

| イベント | 内容 | 用途 |
|---------|------|------|
| U | 中間認識結果（発話中に更新） | UIのリアルタイム表示 |
| A | 最終認識結果（発話区間確定） | AIへの入力トリガー |

**AI呼び出しは必ずAイベント**をトリガーにします。Uイベントは途中結果なのでテキストが変わり続けます。

### 5. 音声向けシステムプロンプトの設計

音声で読み上げることを前提にすると、テキストチャット向けの回答形式をそのまま使うと違和感が生じます。

避けるべき表現:
- `・` `-` などの箇条書き記号
- `①②③` などの丸数字
- コードブロックやテーブル
- `（括弧）` が多用された文体

推奨する表現:
- 「まず〜、次に〜、最後に〜」で列挙する
- 「〜ですね。〜ということになります。」のような口語調

---

## 専門領域モデルへの切り替え

grammarFileNamesを変えるだけで専門エンジンに切り替えられます。

```javascript
// 汎用日本語
const sCommand = `s LSB16K -a-general authorization=${APPKEY}`;

// 医療・電子カルテ向け（医療用語・薬品名に強い）
const sCommand = `s LSB16K -a-medgeneral authorization=${APPKEY}`;

// コールセンター向け（電話音質に対応）
const sCommand = `s LSB16K -a-callcenter authorization=${APPKEY}`;

// 金融・ビジネス向け
const sCommand = `s LSB16K -a-bizfinance authorization=${APPKEY}`;
```

医療記録システムでは「右季肋部痛（うみぎろくぶつう）」「狭心症（きょうしんしょう）」などの専門用語が正確に認識されるようになり、そのまま Claude に渡すと電子カルテの下書き生成や SOAP ノート構造化が実現できます。

---

## 発展的な機能

### 話者ダイアライゼーション（話者分離）

WebSocket/同期HTTPでは、`segmenterProperties` パラメータで話者分離を有効にできます。

```javascript
// sコマンドに追加
const sCommand = [
  `s LSB16K -a-general`,
  `authorization=${APPKEY}`,
  `segmenterProperties=useDiarizer=1`,          // 話者分離ON
  `diarizationMinSpeaker=2`,                     // 最小話者数
  `diarizationMaxSpeaker=5`,                     // 最大話者数
].join(' ');
```

レスポンスの `results[0].tokens` に `speakerId` が付与されます。これを使うと「話者Aが言ったこと」「話者Bが言ったこと」を分けて Claude に渡し、構造化した議事録を生成できます。

### ユーザー辞書（カスタム語彙）

社内固有の製品名・プロジェクト名はユーザー辞書APIで登録することで認識精度を上げられます。

```bash
# 単語登録 REST API
curl -X PUT "https://acp-api.amivoice.com/v1/profile-words" \
  -H "Authorization: Bearer ${APPKEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "profileId": "my-project",
    "words": [
      {
        "written":  "プロジェクトフェニックス",
        "spoken":   "ぷろじぇくとふぇにっくす",
        "class":    "固有名詞"
      }
    ]
  }'
```

sコマンドで `profileId=my-project` を指定すると登録語が有効になります。ユーザーごとに異なる辞書を持たせることも可能です。

### 感情解析（非同期HTTPのみ）

非同期HTTP APIには、音声から感情（喜び・怒り・ストレスなど20パラメータ）を分析するオプションがあります。コールセンターの応対品質評価や、社員のメンタル状態モニタリングなどに活用されています。感情解析はリアルタイムWebSocketではなくバッチ処理での利用になります。

---

## まとめ

AmiVoice API × Claude API で構築した音声対話システムで解決できた課題を整理します。

| 課題 | 対策 |
|------|------|
| 日本語専門用語の誤認識 | AmiVoice専門領域モデルに切り替え |
| 応答の不自然さ（箇条書きなど）| 音声前提のシステムプロンプト設計 |
| 認識中の無応答感 | Uイベントで中間テキストをリアルタイム表示 |
| APIキーの漏洩リスク | Node.jsプロキシでブラウザから隠蔽 |
| バイトオーダー起因の誤認識 | LE環境では LSB16K を明示的に指定 |

特に注意が必要なのは**sコマンドの正確なフォーマット**と**audioFormat（LE/BE）の指定**です。この2点を間違えると認識が全く動かないか、認識精度が著しく低下します。

ハンズフリー操作が必要な工場・医療・物流の現場では、キーボードが使えない状況でも音声でAIに質問できるインターフェースが実用的です。本記事の実装をベースに、専門領域モデルや話者分離など用途に合わせた拡張をぜひ試してみてください。

---

## 参考

- [AmiVoice API マニュアル](https://docs.amivoice.com/amivoice-api/manual/)
- [WebSocket インターフェース概要](https://docs.amivoice.com/amivoice-api/manual/reference/websocket/)
- [sコマンドパケット仕様](https://docs.amivoice.com/amivoice-api/manual/reference/websocket/command/s-command-packet/)
- [音声認識エンジン一覧](https://docs.amivoice.com/en/amivoice-api/manual/engines/)
- [話者ダイアライゼーション](https://docs.amivoice.com/amivoice-api/manual/user-guide/function/speaker-diarization/)
- [ユーザー辞書API](https://docs.amivoice.com/en/amivoice-api/manual/user-dictionary-api/)
- [AmiVoice API クライアントライブラリ（GitHub）](https://github.com/advanced-media-inc/amivoice-api-client-library)
- [Anthropic Claude API リファレンス](https://docs.anthropic.com/en/api/getting-started)
