# GitHub Copilot モデルの選び方と設定によるトークン節約

最終更新: 2026-05-23

## プレミアムリクエストの仕組み

GitHub Copilotは2025年以降、**プレミアムリクエスト（Premium Requests）**という概念を採用している。

- **ベースモデル（無制限）**: GPT-4o mini / GPT-4.1 mini など。プレミアムリクエストを消費しない
- **プレミアムモデル（月の上限あり）**: 高性能モデルはプレミアムリクエストとしてカウントされる
- モデルごとに消費倍率（multiplier）が異なる

### プランごとの月間プレミアムリクエスト上限

| プラン | 月間上限 | 価格 |
|--------|---------|------|
| Copilot Free | 50回 | 無料 |
| Copilot Pro | 300回 | $10/月 |
| Copilot Pro+ | 1,500回 | $39/月 |

## モデルごとの消費倍率（2025〜2026年時点）

| モデル | 消費倍率 | 主な特徴 |
|--------|---------|---------|
| GPT-4o mini | 0倍（無制限） | 高速・軽量。インライン補完に最適 |
| GPT-4.1 mini | 0倍（無制限） | GPT-4.1の軽量版 |
| GPT-4o | 1倍 | デフォルトモデル。汎用 |
| GPT-4.1 | 1倍 | コーディング特化 |
| Claude 3.5 Sonnet | 1倍 | 長文・コーディング・説明に強い |
| Claude 3.7 Sonnet | 1倍 | 高度なコーディング向け |
| Gemini 2.0 Flash | 1倍 | 大コンテキスト対応（100万トークン） |
| o4-mini | 1〜2倍 | 推論特化。数学・アルゴリズム向き |
| o3-mini | 1〜2倍 | 数学・推論タスク向き |
| Gemini 2.5 Pro | 高倍率 | 最高性能Gemini |
| **o3** | **10倍** | 最高推論性能。日常使い非推奨 |

> 倍率は概算。最新情報は公式ドキュメント（about-premium-requests）で確認すること。

## タスク別の推奨モデル

| タスク | 推奨モデル | 消費 |
|--------|-----------|------|
| インライン補完（日常） | GPT-4o mini / GPT-4.1 mini | 無制限 |
| 一般的なコード質問 | GPT-4o | 1倍 |
| リファクタリング・設計 | GPT-4.1 / Claude 3.5 Sonnet | 1倍 |
| 大量コードの読み取り | Gemini 2.0 Flash | 1倍 |
| 数学・アルゴリズム問題 | o4-mini | 1〜2倍 |
| Agent Mode全般 | GPT-4.1 / Claude 3.7 Sonnet | 1倍 |
| 最高品質が必要な重要タスク | o3 / Gemini 2.5 Pro | 10倍以上 |

## Agent Modeとトークン消費

Agent Modeは1タスクで複数回のリクエストを内部発行するため、通常チャットより大量消費する。

### Copilot Pro（月300回）でのコスト試算

| シナリオ | GPT-4.1（1倍） | o3（10倍） |
|---------|--------------|-----------|
| 中規模リファクタリング | 10〜20回消費 | 100〜200回消費 |
| 大規模機能追加 | 30〜50回消費 | 300〜500回（**上限超過**） |

**o3をAgent Modeで使うと月の上限がすぐ枯渇する。**

## VSCode設定による節約

### 不要な言語の補完を無効化

```json
{
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": false,
    "yaml": false,
    "scminput": false
  }
}
```

### 補完を手動トリガーに変更

```json
{
  "github.copilot.editor.enableAutoCompletions": false
}
```

`Alt+\` を押したときのみ補完が発動し、自動補完による無駄なリクエストを減らせる。

## 節約チェックリスト（優先度順）

- [ ] インライン補完のモデルをGPT-4o miniに変更（無制限になる）
- [ ] o3モデルの使用を月1〜2回以内に制限
- [ ] Agent ModeにはGPT-4.1またはClaude 3.5 Sonnetを使う
- [ ] 不要な言語のインライン補完を無効化
- [ ] 新しいタスクごとに新規チャットを開始する

## 参考リンク

- https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests
- https://docs.github.com/en/copilot/using-github-copilot/ai-models/changing-the-ai-model-for-copilot-chat
- https://docs.github.com/en/copilot/about-github-copilot/subscription-plans-for-github-copilot
