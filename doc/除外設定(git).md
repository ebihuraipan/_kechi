<!-- source: reports/copilotignore-effectiveness_20260523-1812.md -->
<!-- section: .gitignoreとの関係（重要） -->

## .gitignoreによる除外

Copilot は Git で追跡されていないファイルを**コンテキストから自動除外**する。
`.gitignore` に含まれるファイルは Copilot が自動除外するが、そのファイルをエディタで開いている・選択している状態では除外設定が無視されコンテキストに含まれる。
設定コストが低く効果が大きいため、新規プロジェクト作成時に最初に確認すべき項目。

効果★★★
