# .github/instructions/ の applyTo によるトークン効率化

作成日: 2026-05-24
参照: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot

---

## 概要

GitHub Copilot にはリポジトリ固有の指示を与える仕組みが2種類ある。従来の `.github/copilot-instructions.md` は全リクエストに常時差し込まれるのに対し、`.github/instructions/*.instructions.md` は `applyTo` フロントマターで対象ファイルパターンを絞り込め、マッチしたときだけコンテキストに含まれる。これにより、関係のない指示がトークンを消費するムダを排除できる。

---

## 2種類のリポジトリカスタム指示

### copilot-instructions.md（全件適用）

- パス: `.github/copilot-instructions.md`
- 挙動: リポジトリ内で作業する**すべてのリクエスト**に自動差し込み
- 問題: ファイルが大きくなるほど毎回のトークン消費が増加する
- 用途: プロジェクト全体に必ず適用したいごく短い共通指示のみに留めるべき

### instructions/*.instructions.md（条件適用）

- パス: `.github/instructions/任意の名前.instructions.md`
- 挙動: `applyTo` で指定したパターンに**一致したときだけ**差し込まれる
- 効果: 関係ないリクエストではスキップ → トークン消費ゼロ
- ファイル名規則: 必ず `.instructions.md` で終わること
- 対応バージョン: VS Code 1.99以降

---

## applyTo フロントマターの構文

```yaml
---
applyTo: "対象ファイルのグロブパターン"
---
ここに指示内容を書く
```

### グロブパターン例

| パターン | 対象 |
|---|---|
| `"**/*.rb"` | 全Rubyファイル |
| `"app/models/**"` | app/models以下の全ファイル |
| `"**/*.ts,**/*.tsx"` | TypeScript/TSXファイル（カンマ区切りで複数指定） |
| `"**/*.test.*,**/*.spec.*"` | テストファイル全般 |
| `"db/**,**/migrations/**"` | DBマイグレーション関連 |
| `"**"` または `"*"` | 全ファイル（copilot-instructions.md と同等になる） |

`applyTo` を省略した場合も全リクエストに適用される（全件適用と同じ挙動になる点に注意）。

---

## トークン節約の仕組み

Copilot はリクエスト送信時に現在編集中のファイルパスを取得し、各 `.instructions.md` の `applyTo` パターンと照合する。マッチしなかったファイルはコンテキストに含めない。

例: `app/main.py`（Pythonファイル）を編集中

```
ruby-models.instructions.md  → applyTo: "app/models/**/*.rb" → 不一致 → スキップ
frontend.instructions.md     → applyTo: "**/*.ts,**/*.tsx"   → 不一致 → スキップ
python.instructions.md       → applyTo: "**/*.py"            → 一致   → 差し込み
```

---

## 実践的な分割パターン

```
.github/
├── copilot-instructions.md          # 最小共通指示のみ
└── instructions/
    ├── ruby-models.instructions.md  # applyTo: "app/models/**/*.rb"
    ├── frontend.instructions.md     # applyTo: "**/*.ts,**/*.tsx,**/*.vue"
    ├── testing.instructions.md      # applyTo: "**/*.test.*,**/*.spec.*"
    └── database.instructions.md    # applyTo: "db/**,**/migrations/**"
```

---

## 注意点

- `copilot-instructions.md` と `instructions/` は**併用可能**。一致した場合は両方がコンテキストに含まれる
- 各 instructions ファイル自体のサイズも簡潔に保つこと（ヒット時のトークン量に直結）
- ファイルが増えすぎると管理コストが増える。5〜10ファイル程度が目安
- VS Code で使うには設定 `github.copilot.chat.codeGeneration.useInstructionFiles` が `true`（デフォルト）であること
