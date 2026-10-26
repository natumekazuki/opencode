# Subagent Review Boundary

委譲前に読む。OpenCode V2の `subagent` toolを使い、対象、必要な結果、権限、完了条件と必要な参照先を依頼へ渡す。子はfresh contextで動くため履歴継承を前提にしない。全規約を複製せず、必要な文書を解決済みの実pathで渡す。

## 担当

| agent | 用途 |
| --- | --- |
| general | 通常の調査・設計・実装・debug |
| light | 進め方・手順・期待結果がほぼ確定した抽出・実装・照合 |
| judgment | 契約間の矛盾、設計選択、難しい反例等の高度判断 |
| read-only | 調査・review。編集・shell・MCP・再委譲なし |
| explore | file・code・Webの読取探索。編集・shell・MCP・再委譲なし |

model IDと開始値は[README](../../README.md#モデルと権限)と各定義を参照する。割当は初期運用配分であり性能等価の保証ではない。高度判断への委譲に軽量modelの失敗を要求しない。範囲が小さいことだけを理由に未解決の設計や安全性の判断をlightへ押し込まない。

通常の子は再委譲せず、親が担当を選び結果を統合する。起動時の実効agent/modelを確認し、model指定だけでpermissionも引き継がれると推測しない。指定modelが使えない場合は報告し、承認なしに別課金経路へ切り替えない。

## 依頼と結果

reviewでは主要動作、後続の依存、[開発・review・統合](../guides/development.md)の統合基準、非ブロッカー全件修正を条件にしないことを渡す。共有判断に関わる場合は採用済み方針・適用状況・未合意事項と[設計判断・文書化](../guides/design-decisions.md)も渡す。調査・草案作成の委譲を方針採用や操作権限の拡張と解釈しない。

test価値reviewは[review-test-value](../../skills/review-test-value/SKILL.md)の全recordをread-onlyへ渡す。親が必要なtest実行・diff抽出・結果保存を行い、子は許可されたtoolでproduction/fixture/oracleを確認する。read-only子へ一時fileの保存やshell実行を依頼しない。必要な情報がtool制約で取得できなければ、親へ不足を返す。

同じworking treeでは担当fileと生成物の所有を明示し、他者が並行作業中であること、他者の編集を巻き戻さないことを伝える。必要な隔離がある場合だけworktreeを使う。結果は根拠・確認範囲・未解決を親へ返し、専用の管理基盤や結果台帳を新設しない。親は指摘の採否・対応時期・統合判断を行い、必要な結果記録をSessionFolderへ保存する。
