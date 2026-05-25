# .copilotignoreの利用がトークン数節約に効果的か調査

## エグゼクティブサマリー

`.copilotignore`はトークン節約に効果があるが、**条件付き**。`.gitignore`に含まれるファイルはCopilotが自動除外するため、`.copilotignore`の追加価値は「gitには追跡させたいが、Copilotには見せたくない」ファイルに限られる。また効果が出るのは主に`@workspace`使用時であり、Tab補完への影響は軽微。

---

## `.copilotignore`とは

`.gitignore`と同書式で、Copilotがコンテキストとして読み込むファイルを除外するファイル。リポジトリルートに配置する。

```gitignore
# .copilotignore
fixtures/
generated-docs/
*.secret
```

---

## `.gitignore`との関係（重要）

**CopilotはGitで追跡されていないファイルを自動的にコンテキストから除外する。**

つまり`.gitignore`に含まれているファイルは、`.copilotignore`に書かなくてもCopilotはすでに無視している。

| ファイルの状態 | Copilotの動作 |
|--------------|--------------|
| `.gitignore`に含まれる | 自動除外（`.copilotignore`不要） |
| Gitで追跡中・`.copilotignore`に含まれる | Copilotが除外 |
| Gitで追跡中・どちらにも含まれない | Copilotがコンテキストに含める |

**`.copilotignore`が追加価値を発揮するケース:**

- テストfixture（gitで管理したいが、Copilotの提案に影響させたくない）
- 生成ドキュメント（`docs/generated/`など）
- 機密データファイル（念押しで除外したい場合）
- 特定のプロジェクトメンバーの環境固有ファイル

---

## 効果が出る場面・出ない場面

### 効果大

- **`@workspace`使用時**: `.copilotignore`対象ファイルは検索対象から除外される。ファイル数が多いほど節約効果が大きい
- **Copilot Chatの自動コンテキスト収集**: 関連ファイルを自動で探す処理からも除外される

### 効果小〜なし

- **Tab補完（Ghost Text）**: カーソル周辺のコードのみを参照するため、`.copilotignore`の影響はほぼない
- **`#file:`で明示参照したファイル**: `.copilotignore`に含まれていても明示指定すれば読み込まれる（設計上の仕様）

---

## 設定例

```gitignore
# .copilotignore

# テストデータ（gitで管理するがCopilotには不要）
tests/fixtures/
tests/__snapshots__/

# 生成ドキュメント
docs/generated/

# 機密・環境依存（.gitignoreにも入れるべきだが念押し）
*.secret
.env.local

# 大きなデータファイル
data/raw/
*.csv
*.sql
```

---

## 結論

`.copilotignore`の節約効果をまとめると：

1. **`@workspace`を使う場合**: 効果大。不要ファイルを除外することでCopilotの検索コストが減る
2. **`@workspace`を使わない通常の補完・Chat**: 効果は限定的
3. **`.gitignore`で管理済みのファイル**: `.copilotignore`に書かなくても自動除外されるため二重管理不要

**推奨**: `@workspace`を多用するなら設定価値あり。Tab補完中心ならコストパフォーマンスは低め。設定コストは一度きりなので、使う場合は積極的に`.gitignore`に入れられないファイルを追加する。
