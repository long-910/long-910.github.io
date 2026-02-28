---
title: Privacy Policy
icon: fas fa-shield-alt
order: 7
---

<div id="lang-switch" style="margin-bottom:1.5rem">
  <button id="btn-en" onclick="switchLang('en')" style="margin-right:.5rem;padding:.3rem .8rem;border-radius:4px;border:2px solid;cursor:pointer">🇺🇸 English</button>
  <button id="btn-ja" onclick="switchLang('ja')" style="margin-right:.5rem;padding:.3rem .8rem;border-radius:4px;border:1px solid;cursor:pointer">🇯🇵 日本語</button>
  <button id="btn-zh" onclick="switchLang('zh')" style="padding:.3rem .8rem;border-radius:4px;border:1px solid;cursor:pointer">🇨🇳 中文</button>
</div>

<div id="content-en" class="lang-content" markdown="1">

_Last updated: 2026-02-28_

This Privacy Policy describes how this website (https://910.jp) and the VS Code extensions published by **long-kudo** handle your information.

---

## VS Code Extensions

The VS Code extensions published by long-kudo — **VSCode Claude Status** and **VSCode View Charset** — do not collect, store, or transmit any personal data.

- No personal information is collected.
- No usage analytics or telemetry are sent from within the extensions.
- No external API calls are made, except for the minimum required for the extension's core functionality (e.g., reading local VS Code session state).

---

## This Website

**Analytics**

This website uses [Google Analytics](https://marketingplatform.google.com/about/analytics/) to collect anonymized usage statistics (page views, referrers, device type). This data is used solely to understand site traffic and improve content. No personally identifiable information is collected. You can opt out via [Google Analytics Opt-out](https://tools.google.com/dlpage/gaoptout).

**Comments**

Posts on this site use [utterances](https://utteranc.es/) for comments, which are stored as GitHub Issues. Leaving a comment requires authentication via GitHub. Please refer to [GitHub's Privacy Statement](https://docs.github.com/en/site-policy/privacy-policies/github-privacy-statement) for details on how GitHub handles your data.

---

## Contact

If you have any questions about this privacy policy, please contact via [GitHub](https://github.com/long-910).

</div>

<div id="content-ja" class="lang-content" style="display:none" markdown="1">

_最終更新日: 2026-02-28_

このプライバシーポリシーは、本ウェブサイト（https://910.jp）および VS Code publisher **long-kudo** が公開する VS Code 拡張機能が、お客様の情報をどのように扱うかを説明するものです。

---

## VS Code 拡張機能

long-kudo が公開する VS Code 拡張機能（**VSCode Claude Status** および **VSCode View Charset**）は、個人データの収集・保存・送信を一切行いません。

- 個人情報は収集しません。
- 拡張機能から使用状況の分析やテレメトリーを送信しません。
- 拡張機能のコア機能に必要な最小限のもの（例: ローカルの VS Code セッション状態の読み取り）を除き、外部 API への通信は行いません。

---

## 本ウェブサイト

**アクセス解析**

本ウェブサイトは、匿名化された利用統計情報（ページビュー、参照元、デバイス種別）の収集に [Google Analytics](https://marketingplatform.google.com/about/analytics/) を使用しています。このデータはサイトのトラフィックを把握しコンテンツを改善するためにのみ使用します。個人を特定できる情報は収集されません。[Google Analytics オプトアウト](https://tools.google.com/dlpage/gaoptout)からオプトアウトできます。

**コメント**

本サイトの記事コメントには [utterances](https://utteranc.es/) を使用しており、コメントは GitHub Issue として保存されます。コメントを残すには GitHub 認証が必要です。GitHub によるデータの取り扱いについては、[GitHub のプライバシーに関する声明](https://docs.github.com/en/site-policy/privacy-policies/github-privacy-statement)をご参照ください。

---

## お問い合わせ

このプライバシーポリシーに関するご質問は、[GitHub](https://github.com/long-910) からお問い合わせください。

</div>

<div id="content-zh" class="lang-content" style="display:none" markdown="1">

_最后更新日期：2026-02-28_

本隐私政策说明本网站（https://910.jp）及 VS Code 发布者 **long-kudo** 所发布的 VS Code 扩展如何处理您的信息。

---

## VS Code 扩展

long-kudo 发布的 VS Code 扩展（**VSCode Claude Status** 和 **VSCode View Charset**）不收集、存储或传输任何个人数据。

- 不收集任何个人信息。
- 扩展程序不发送任何使用分析或遥测数据。
- 除扩展核心功能所需的最低限度外（例如读取本地 VS Code 会话状态），不进行任何外部 API 调用。

---

## 本网站

**统计分析**

本网站使用 [Google Analytics](https://marketingplatform.google.com/about/analytics/) 收集匿名使用统计信息（页面浏览量、来源渠道、设备类型）。这些数据仅用于了解网站流量并改善内容，不收集任何可识别个人身份的信息。您可通过 [Google Analytics 退出工具](https://tools.google.com/dlpage/gaoptout) 选择退出。

**评论**

本网站文章评论使用 [utterances](https://utteranc.es/)，评论以 GitHub Issue 的形式存储。发表评论需通过 GitHub 进行身份验证。有关 GitHub 如何处理您的数据，请参阅 [GitHub 隐私声明](https://docs.github.com/en/site-policy/privacy-policies/github-privacy-statement)。

---

## 联系方式

如对本隐私政策有任何疑问，请通过 [GitHub](https://github.com/long-910) 与我们联系。

</div>

<script>
var LANGS = ['en', 'ja', 'zh'];
function switchLang(lang) {
  LANGS.forEach(function(l) {
    document.getElementById('content-' + l).style.display = 'none';
    var btn = document.getElementById('btn-' + l);
    btn.style.borderWidth = '1px';
    btn.style.fontWeight  = 'normal';
  });
  document.getElementById('content-' + lang).style.display = 'block';
  var active = document.getElementById('btn-' + lang);
  active.style.borderWidth = '2px';
  active.style.fontWeight  = 'bold';
  try { localStorage.setItem('privacyLang', lang); } catch(e) {}
}
(function() {
  var saved = '';
  try { saved = localStorage.getItem('privacyLang'); } catch(e) {}
  var nav = (navigator.language || '').toLowerCase();
  var detected = nav.startsWith('zh') ? 'zh' : nav.startsWith('ja') ? 'ja' : 'en';
  switchLang(saved || detected);
})();
</script>
