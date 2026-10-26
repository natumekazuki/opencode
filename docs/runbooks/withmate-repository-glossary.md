# WithMate Repository Glossary 接続

repository用語の意味確認・登録は[WithMate利用方針](../guides/withmate.md)とruntime-managed `withmate-glossary` Skillを参照する。本書は接続・配置の診断用であり、tool契約を複製しない。

## 接続とSkill

OpenCode対応WithMateは `withmate-glossary` をmanaged HTTP MCPとしてturnごとに供給する。endpointは `/mcp/glossary`。URL・短命tokenはhostが渡し、native configへ登録しない。Memoryと共通のbinding境界は[Memory／Character runbook](withmate-character-context.md)を参照する。

WithMateのSkill同期先を `~/.config/opencode/skills` に合わせる。任意の別folderへ同期しただけではOpenCodeのnative discovery対象にはならない。配布された `skills/withmate-glossary/` はWithMate管理とし、このrepositoryでは追跡しない。managed markerがない同名folderを無断で上書き・forkしない。配置と更新はWithMate側の配布契約に従う。

## 操作境界

現在の `list_targets` とSettingsが許可するrepositoryだけを操作する。Additional Directoriesへ追加されたpathをGlossaryのwrite targetへ自動拡張しない。`.withmate/glossary.yaml` のschema・revision・更新手順は現在のmanaged Skillとtool schemaで確認する。

結果不明時はread-backで現状を確認し、revision競合を古い入力で再実行しない。検証失敗・権限拒否・target不一致をCLI fallbackで迂回しない。read/search/validate/create/create-batch/updateの継続許可とdeleteの個別承認はguideに従う。

接続確認は必要なread-only操作までとし、無用なentryを作らない。未接続・未配置・未確認を区別して報告する。手動STDIO診断のcommandは `withmate-glossary mcp-server` だが、現runtimeの許可とbindingが必要であり標準の自動代替にはしない。
