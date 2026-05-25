# 拡張子毎にインライン補完させない方法まとめて

## 概要

GitHub Copilot のインライン補完（幽霊テキスト）は、VSCode の `settings.json` で言語IDごとにオン/オフを制御できる。補完が不要なファイルタイプを無効化するだけでAPIリクエスト数を減らせるため、トークン・クレジット節約の手段として有効。

---

## 設定方法

`github.copilot.enable` に言語IDをキーとしたオブジェクトを指定する。

### 基本構文

```json
// .vscode/settings.json または ユーザー settings.json
{
  "github.copilot.enable": {
    "*": true,
    "markdown": false,
    "plaintext": false
  }
}
```

`*` はワイルドカードで「上記以外のすべて」を意味する。

---

## パターン A: デフォルト有効・特定のみ無効（ピンポイント除外）

補完は基本使いたいが、特定のファイルタイプだけ切りたい場合。

```json
{
  "github.copilot.enable": {
    "*": true,
    "plaintext": false,
    "markdown": false,
    "yaml": false,
    "json": false,
    "jsonc": false,
    "csv": false,
    "log": false,
    "xml": false,
    "toml": false,
    "dotenv": false,
    "sql": false
  }
}
```

---

## パターン B: デフォルト無効・特定のみ有効（ホワイトリスト）

節約を最優先にして、補完が本当に必要な言語だけ許可する場合。クレジット消費を最も抑えられる。

```json
{
  "github.copilot.enable": {
    "*": false,
    "javascript": true,
    "javascriptreact": true,
    "typescript": true,
    "typescriptreact": true,
    "python": true,
    "rust": true,
    "go": true
  }
}
```

---

## 主な言語IDと対応拡張子

| 拡張子 | 言語ID |
|---|---|
| `.js` | `javascript` |
| `.jsx` | `javascriptreact` |
| `.ts` | `typescript` |
| `.tsx` | `typescriptreact` |
| `.py` | `python` |
| `.go` | `go` |
| `.rs` | `rust` |
| `.md` | `markdown` |
| `.txt` | `plaintext` |
| `.yaml` / `.yml` | `yaml` |
| `.json` | `json` |
| `.jsonc` | `jsonc` |
| `.toml` | `toml` |
| `.xml` | `xml` |
| `.html` | `html` |
| `.css` | `css` |
| `.scss` | `scss` |
| `.sql` | `sql` |
| `.sh` | `shellscript` |
| `.env` | `dotenv` |
| `.log` | `log` |
| `.csv` | `csv` |

---

## 言語IDの確認方法

VSCode のステータスバー右下に現在の言語名が表示されている。それをクリックすると言語選択ピッカーが開き、「言語IDをコピー」で正確なIDを取得できる。

---

## 設定スコープ

| スコープ | ファイルパス | 適用範囲 |
|---|---|---|
| ユーザー設定 | `~/.config/Code/User/settings.json` など | すべてのプロジェクト |
| ワークスペース設定 | `.vscode/settings.json` | そのプロジェクトのみ |

プロジェクトによって補完が必要かどうか変わる場合は、ワークスペース設定で上書きするのが有効。

---

## 注意事項

- この設定が効くのは**インライン補完（幽霊テキスト）のみ**。Copilot Chat・エージェントモードには影響しない。
- `.copilotignore` でファイルをコンテキストから除外するのとは別の設定。インライン補完は `.copilotignore` では制御できない。
- 設定変更後にVSCodeの再起動は不要。即時反映される。

---

## トークン節約の観点からの推奨

設定・データファイル系やドキュメント系は補完の恩恵が薄いうえ、頻繁に編集するとAPIリクエストが積み重なる。以下を無効化候補の優先リストとして参考にする。

1. `plaintext` / `markdown` - ドキュメント、READMEなど
2. `yaml` / `toml` / `json` / `jsonc` - 設定ファイル全般
3. `dotenv` - 環境変数ファイル
4. `csv` / `log` / `xml` - データ・ログファイル
5. `sql` - クエリファイル（定型的なものが多い場合）
