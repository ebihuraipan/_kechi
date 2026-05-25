# GitHub Copilot の推論エフォート（Reasoning Effort）設定についてレポートを作って。

作成日: 2026-05-24
出典: [GitHub Docs - Using reasoning models with GitHub Copilot](https://docs.github.com/en/copilot/using-github-copilot/ai-models/using-reasoning-models-with-github-copilot) / [GitHub Docs - About premium requests](https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests)

---

## 推論エフォートとは

推論エフォート（Reasoning Effort）は、GitHub Copilot の推論モデルが回答を生成する際にどれだけのリソース（計算・思考トークン）を使うかを制御するパラメータ。

エフォートを高くするほど：
- 時間をかけて問題を「考える」ステップが増える
- 思考トークン（thinking tokens）の消費量が増加する
- 精度が上がりやすいが、コストも上がる

エフォートを低くするほど：
- 素早く回答する
- 思考トークンの消費量を抑えられる
- 単純なタスクでは十分な精度を保てる

### 対応モデル

推論エフォートは**推論モデル（Reasoning Models）のみ**対応。通常の補完モデルには設定できない。

| モデル | 推論エフォート対応 | デフォルトエフォート |
|--------|-------------------|---------------------|
| o3 | Low / Medium / High | Medium |
| o4-mini | Low / Medium / High | Medium |
| o3-mini | Low / Medium / High | Medium |
| GPT-4o | 非対応 | - |
| Claude Sonnet 4.5 | 非対応（通常版）| - |

> x-High は一部モデル・環境で提供される場合がある。

---

## 推論エフォートの設定方法

### モデルピッカーから設定（VS Code）

1. Copilot Chat パネルを開く
2. チャット入力欄の左にある **モデルピッカー** をクリック
3. `o3` または `o4-mini` などの推論モデルを選択する
4. モデル名の横に **「Reasoning Effort」ドロップダウン** が表示される
5. `Low` / `Medium` / `High` から選択する

GitHub.com の Copilot Chat（ブラウザ版）でも同様の UI が提供されている。

### settings.json での設定

2026年5月時点では、VS Code 安定版の `settings.json` に推論エフォートを固定するための公式の設定キーは公開されていない。設定は UI 経由（モデルピッカー）で行う。

---

## PRU 課金制（〜2026/5/31）での影響

**エフォートを変えても PRU 消費数は変わらない。**

PRU はリクエスト単位の固定レートで課金される。

| モデル | PRU レート | エフォート影響 |
|--------|-----------|--------------|
| o3 | 10 PRU / リクエスト | エフォートに関係なく固定 |
| o4-mini | 0.33 PRU / リクエスト | エフォートに関係なく固定 |
| GPT-4o | 1 PRU / リクエスト | - |
| Claude Sonnet 4.5 | 1 PRU / リクエスト | - |

PRU 課金制では精度が上がるなら High / x-High にしても損はない。コスト削減するならモデル自体を o3 → o4-mini に変える方が効果的。

---

## トークン従量課金制（2026/6/1〜）での影響

2026年6月1日以降、PRU 固定制からトークンベースの従量課金制に移行予定。

**移行後は推論エフォートのレベルが直接コストに影響する。**

```
Low エフォート  → 思考トークン 少 → コスト 低
Medium エフォート → 思考トークン 中 → コスト 中
High エフォート  → 思考トークン 多 → コスト 高
```

思考トークン（thinking tokens）はモデルが内部で「考える」ために使うトークン。通常のアウトプットトークンとは別に生成され、トークンベース課金では課金対象となる。

---

## タスク別の使い分け指針

| タスク | 推奨エフォート |
|--------|-------------|
| コメント・ドキュメント生成 | Low |
| 変数名変更・単純リファクタリング | Low |
| ユニットテスト生成（単純） | Low |
| 既存コードの説明 | Low |
| バグ修正（単純） | Low〜Medium |
| 新機能の実装（中規模） | Medium |
| コードレビュー・改善提案 | Medium |
| 複雑なアルゴリズムの設計 | High |
| バグの根本原因分析（複雑） | High |
| セキュリティ脆弱性の分析 | High |
| アーキテクチャ検討 | High |

---

## インライン補完（コード補完）はクレジット不消費

公式ドキュメント（about-premium-requests）に明記：

> "Code completions, such as the inline suggestions that appear as you type in your editor, don't count as premium requests."

- **PRU を消費しない**
- **AIクレジットを消費しない**
- ベースサブスクリプションの範囲内で処理される

Copilot Chat はPRU/トークンを消費するが、インライン補完（ghost text）はコスト面での制限なく使える。短いコード生成はチャットではなくインライン補完で行うとコスト削減になる。

---

## まとめ・推奨設定

| 状況 | 推奨 |
|------|------|
| PRU制（〜5/31）でコスト削減したい | エフォートより**モデルを o4-mini へ変更**（o3 の 1/30） |
| PRU制でとにかく精度を上げたい | High / x-High（コスト同じ） |
| トークン制（6/1〜）でコスト削減したい | **Low エフォートをデフォルト**に |
| 日常的なコーディング補助 | インライン補完を積極使用（コスト不消費）|
| 複雑な設計・デバッグ | o3 + High エフォート（必要な時だけ） |

---

*参照:*
- *[About premium requests - GitHub Docs](https://docs.github.com/en/copilot/managing-copilot/monitoring-usage-and-entitlements/about-premium-requests)*
- *[Using reasoning models with GitHub Copilot](https://docs.github.com/en/copilot/using-github-copilot/ai-models/using-reasoning-models-with-github-copilot)*
