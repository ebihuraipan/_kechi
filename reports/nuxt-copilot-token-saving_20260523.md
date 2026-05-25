# Nuxtプロジェクトにおける GitHub Copilot トークン節約の実践ガイド

**作成日**: 2026-05-23
**調査テーマ**: Nuxt プロジェクトでの GitHub Copilot コンテキスト最適化とトークン削減手法

---

## エグゼクティブサマリー

Nuxt プロジェクトは `nuxi dev` / `nuxi build` の実行により大量の自動生成ファイルを生成するため、GitHub Copilot の `@workspace` インデックス上限（2,500 ファイル）を容易に超過する。これにより回答精度が低下し、無関係なファイルがコンテキストに混入するリスクが高まる。

対策の基本方針は「**Nuxt 公式推奨の `.gitignore` を確実に設定し、さらに `.copilotignore` で Git 管理下の不要ファイルを追加除外する**」ことである。加えて、`copilot-instructions.md` の肥大化防止、Agent Mode の制限の把握、`llms.txt` や MCP サーバーの適切な利用が有効な補完手段となる。

---

## @workspace のインデックス上限と Nuxt の問題

VS Code が `@workspace` 用にローカルインデックスを構築する際の上限は **2,500 ファイル** である。この上限を超えると、セマンティック検索から単純なテキスト検索にフォールバックし、回答精度が著しく低下する。

Nuxt プロジェクトはデフォルト状態でこの上限を容易に超過する。問題となる主なディレクトリを以下に示す。

| ディレクトリ | 生成タイミング | 内容 |
|---|---|---|
| `.nuxt/` | `nuxi dev` / `nuxi build` | 型定義・ルート定義・Vite キャッシュ等 |
| `node_modules/` | `npm install` 等 | 依存パッケージ全体 |
| `.output/` | `nuxi build` | Nitro のビルド出力 |
| `.nitro/` | ビルド時 | Nitro のキャッシュ |
| `.cache/` | 実行時 | Nuxt の各種キャッシュ |
| `dist/` | `nuxi generate` | 静的出力 |

これらがインデックス対象に含まれると、`@workspace` を使った質問のコンテキスト品質が低下するだけでなく、無関係なファイルが応答に混入する原因ともなる。

**重要な挙動：** `.gitignore` に含まれるファイルは Copilot がデフォルトで検索対象から除外する。ただし「除外対象ファイルをエディタで開いている、または選択している」状態だと `.gitignore` の指定が無視され、コンテキストに含まれる点に注意が必要である。

---

## .gitignore で除外すべき Nuxt 固有ディレクトリ

Nuxt 公式が推奨する `.gitignore` の設定を必ず行う。これらは `.gitignore` に記載するだけで Copilot の `@workspace` インデックスから自動除外される。

```gitignore
# Nuxt dev/build outputs
.output
.data
.nuxt
.nitro
.cache
dist

# Node dependencies
node_modules

# Logs
logs
*.log

# Misc
.DS_Store

# Local env files
.env
.env.*
!.env.example
```

この設定が抜けているだけで `@workspace` の実効性が大幅に低下するため、新規プロジェクト作成時に最初に確認すべき項目である。

---

## .copilotignore を追加すべき Nuxt 固有ファイル

`.copilotignore` は「Git で管理はするが、Copilot に読ませたくないファイル」を除外するために使用する。`.gitignore` でカバーできないが Copilot のコンテキストには不要なファイルが対象となる。

Nuxt プロジェクトで追加を検討すべき例を以下に示す。

```
# 自動生成された型ファイルが大量にある場合
/types/generated/**

# Storybook や Vitest のスナップショット
**/__snapshots__/**

# テスト用フィクスチャ（大量の HTML/JSON）
/tests/fixtures/**

# ビルド設定の自動生成部分
/server/tsconfig.json
```

`.copilotignore` の効果と制限については別資料（`copilotignore-effectiveness_*.md`）を参照のこと。

---

## copilot-instructions.md の最小化

`.github/copilot-instructions.md` はすべてのリクエストに自動付加されるため、このファイルが肥大化するとそれ自体がトークンコストになる。

### 書かなくてよいもの

- Nuxt の基本的な説明（Copilot はフレームワークの仕様をすでに学習済みである）
- 公式ドキュメントで確認できる一般的な使い方

フレームワーク説明を削除するだけで **20〜23% のトークン削減効果** があるという報告がある。

### 書くべきもの

- このプロジェクト固有の命名規則や構造の例外
- 絶対に使ってはいけないパターン（例：「Options API を使わない」「`<script setup>` のみ使用する」）
- 外部から判断できない依存関係の注意点や制約

記述量は最小限に抑え、Copilot がすでに知っている情報は一切書かないことを原則とする。

---

## Agent Mode のコンテンツ除外の制限

リポジトリの Settings > Copilot > Content exclusion からパスパターンを設定できる機能があるが、**Agent Mode では現時点でこの設定が機能しない**という重要な制限がある。この機能はインライン補完と通常の Copilot Chat にのみ有効である。

Agent Mode を主な使用形態としている場合、Content Exclusion に頼るのは危険である。`.gitignore` と `.copilotignore` による除外設定で対処する必要がある。

Agent Mode を多用する場合の優先対策順位：

1. `.gitignore` の整備（最も確実で広範囲に効く）
2. `.copilotignore` の追加設定（Git 管理下の不要ファイル対策）
3. 質問時にファイルを明示的に指定する（後述）

---

## その他のテクニック

### llms.txt の活用

Nuxt 公式は LLM 向けのドキュメントを提供している。

- `https://nuxt.com/llms.txt`：約 5K トークン。Copilot Chat から参照可能であり、フレームワーク知識を外部化してトークン効率を高められる
- `https://nuxt.com/llms-full.txt`：1M トークン超えのため使用は非推奨

### Nuxt 公式 MCP サーバー

Nuxt が公式に提供する MCP サーバーを活用することで、フレームワーク知識の供給を外部化できる。ただし MCP ツールは登録するだけで **1 ツールあたり 100〜500 トークンのスキーマオーバーヘッド** が発生するため、実際に使用するツールのみ有効化する。

### @workspace および質問粒度の最適化

- 質問の対象を絞る：「このプロジェクト全体で...」より「`/server/api/users.ts` の...」のように対象ファイル・関数を明示する
- `#file:composables/useAuth.ts` のようにファイルを明示的に指定すると余分な検索が省かれ、トークンの無駄遣いを防げる
- `.nuxt/types/` 配下の型ファイルを意図せず開いたまま作業しない（開いているファイルは `.gitignore` の設定を無視してコンテキストに含まれる）

### nuxt.config.ts の整理

Copilot がプロジェクト全体のコンテキスト把握によく参照するファイルである。コメントや未使用設定を削除し、必要最小限に整理しておくことでコンテキスト品質が向上する。

---

## 結論

Nuxt プロジェクトで Copilot のトークン効率を高めるための対策を効果順にまとめる。

1. **Nuxt 公式推奨の `.gitignore` を確実に設定する**（最優先・最大効果）
2. **`.copilotignore` で Git 管理下の不要ファイルを追加除外する**
3. **`copilot-instructions.md` をプロジェクト固有の情報のみに絞り込む**
4. **Agent Mode を多用する場合は Content Exclusion に頼らず上記 1・2 で対処する**
5. **質問時はファイルや関数を明示し、`@workspace` の検索範囲を絞る**
6. **`llms.txt` や MCP サーバーを活用しフレームワーク知識を外部化する**（使い過ぎに注意）

特に `.gitignore` の整備は設定コストが低く効果が大きいため、既存プロジェクトでも即座に適用できる。Agent Mode の Content Exclusion が機能しないという制限は見落としやすい落とし穴であるため、チーム内での周知も重要である。

---

## 参考情報源

- Nuxt 公式 `.gitignore` テンプレート: https://nuxt.com/docs/getting-started/deployment
- GitHub Copilot Content Exclusion 公式ドキュメント: https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-github-copilot-features-in-your-organization/about-content-exclusions-for-github-copilot
- Nuxt llms.txt: https://nuxt.com/llms.txt
- GitHub Copilot カスタム指示ドキュメント: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot
