# GitHub CopilotのPR自動要約機能（Copilot for Pull Requests / Generate summary）を使った場合、トークン・クレジットは誰のサブスクリプションから消費されるか？ボタンを押したユーザーか、PRの作者か、リポジトリオーナーか、組織か。個人プランと組織プランで違いがあるか。調査してまとめて。

調査日: 2026-05-24

## 結論（先に）

**ボタンを押した人（操作実行者）のサブスクリプションから消費される。**
PRの作者・リポジトリオーナー・組織は関係ない。

---

## 1. 機能概要

| 項目 | 内容 |
|------|------|
| 正式名称 | Copilot pull request summaries |
| 旧称 | Copilot for Pull Requests |
| 操作方法 | PR画面の「Generate summary with Copilot」ボタンを押す |
| 処理場所 | GitHubサーバー側（VSCode等クライアント不要） |
| 入力 | PRのdiff全体 |

---

## 2. 誰のクレジットが消費されるか

### 基本原則: 操作者主義

GitHub Copilot のPR要約機能は**「使った人のアカウントから引き落とされる」**モデルを採用している。

| 操作者 | 消費元 |
|--------|--------|
| Copilot Individual / Pro 保有者 | その人の個人サブスクリプション |
| Copilot Business シート保有者 | そのユーザーのシートに紐づく組織サブスクリプション |
| Copilot Enterprise シート保有者 | そのユーザーのシートに紐づく Enterprise サブスクリプション |
| Copilot Free ユーザー | そのユーザーの月次無料枠 |
| Copilot 未加入ユーザー | 機能自体が動作しない（エラー） |

### 重要: PRの作者・オーナーは無関係

- 他人のリポジトリのPRで「Generate summary」を押しても、**自分のクレジットが消費される**
- PRを作った人のクレジットは消費されない
- リポジトリオーナーのクレジットも消費されない

---

## 3. プランごとの違い

### 個人プラン（Copilot Individual / Pro）

- 個人サブスクリプション保有者が押した分だけ消費
- 月次上限があるプランではその枠内で消費
- 機能の有効/無効の管理は個人設定から（Settings → Copilot）

### 組織プラン（Copilot Business）

- **組織管理者が機能を有効化する必要がある**
  - 場所: Organization Settings → Copilot → Policies → "Copilot pull request summaries"
  - ポリシー選択肢: Enabled / Disabled / No policy（Enterprise ポリシーに従う）
- 有効化後、シートを持つ組織メンバーが使った分が組織の費用に計上される
- シートを持たないメンバーは使用不可

### Enterprise プラン（Copilot Enterprise）

- Enterprise レベルと Organization レベルの2段階でポリシー設定が可能
- Enterprise 管理者がデフォルトを設定し、配下の組織が上書きできる（ポリシー設定による）
- 消費主体はやはり「ボタンを押したユーザーのシート」に紐づく

### Copilot Free

- 2024年12月以降に導入された無料枠
- PR要約機能が月次上限（chat turns等）に含まれるかは限定的な記載のみ
- 実質的な主要ユーザーは Pro / Business / Enterprise

---

## 4. トークン消費の特性

### VS Codeエージェントモードとの違い

| 比較項目 | PR要約 | エージェントモード |
|----------|--------|-----------------|
| 処理場所 | GitHubサーバー | クライアント側 |
| tool callループ | なし | あり（乗数的に増える） |
| 消費の予測しやすさ | 高い（diff量に比例） | 低い（ループ次第） |

### 消費量に影響する要因

- **diffのサイズ**: 変更行数が多いPRほどトークン消費が増える（diff全体がコンテキストに入る）
- GitHub は具体的なトークン数を非公開としている
- 大規模なPR（数百ファイル変更など）は消費が大きい可能性がある

---

## 5. ケチるための実践ポイント

- **Copilot を持っていないチームメンバーに押させない**（そもそも機能しないが、誤解を防ぐ）
- **組織プランでは管理者がポリシーをDisabledにすることで機能を封鎖できる**（不要なら無効化）
- **差分が巨大なPRでは使わない**（small PRに限定すると消費を抑えられる）
- 自分のCopilotクレジットが枯渇しそうな月末は、他人のリポジトリのPRで「試しに押す」のを避ける

---

## 参考

- [GitHub Docs: Using Copilot pull request summaries](https://docs.github.com/en/copilot/using-github-copilot/using-copilot-pull-request-summaries)
- [GitHub Docs: Managing Copilot policies as an organization owner](https://docs.github.com/en/copilot/managing-copilot/managing-github-copilot-in-your-organization/managing-copilot-for-your-enterprise/managing-policies-and-features-for-copilot-in-your-enterprise)
