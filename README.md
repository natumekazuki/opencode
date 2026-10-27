# OpenCode 個人設定

OpenCode V2の規約・Skill・agent・設定例を管理する。配置先は `~/.config/opencode/`。Codexを共通方針の基本の正本とし、このrepository単独で利用できる内容に適応する。

## 配置と導入

| 配置 | 責務 |
| --- | --- |
| `AGENTS.md` | 常時原則と条件付き参照の入口 |
| `docs/guides/` | 開発・設計判断・test・UI・WithMateの共通方針 |
| `docs/architecture/subagent-workspace.md` | 委譲と担当の選択 |
| `docs/runbooks/` | WithMateの接続・診断 |
| `skills/` | 自前で保守する6 Skillと必要なscript・reference・license |
| `agents/` | light・judgment・read-onlyのV2定義 |
| `opencode.example.jsonc` | モデルと権限の配布用開始値 |
| `opencode.jsonc` | Git管理しない端末local実設定 |

OpenCode CLI 2.1.4を対象とする。既存の `opencode.json` / `opencode.jsonc` があれば設定例から必要なkeyだけを反映し、認証・MCP・project設定を一括上書きしない。どちらもなければ次で設定例を適用できる。

```powershell
$configRoot = Join-Path ([Environment]::GetFolderPath('UserProfile')) '.config/opencode'
if ((Test-Path -LiteralPath "$configRoot/opencode.json") -or (Test-Path -LiteralPath "$configRoot/opencode.jsonc")) {
    throw '既存の実設定へ必要な項目だけ反映してください'
}
Copy-Item -LiteralPath "$configRoot/opencode.example.jsonc" -Destination "$configRoot/opencode.jsonc"
opencode debug config
opencode debug agents
```

設定はglobal・project・host等でmergeされる。新しいsessionで読込元、実効agent/model/permissionを確認する。既存sessionの選択済みmodelは既定変更だけでは切り替わらない。詳細は[公式V2 config](https://opencode.ai/v2/docs/config/)と[agents](https://opencode.ai/v2/docs/agents/)を参照する。

この場所での編集はlive配置へ即時影響する。別checkoutの配布元編集、local適用、master統合、公開を区別する。共通guideはAGENTS指定の固定pathから読み、別checkoutをruntime配置の代用にしない。

## モデルと権限

| 担当 | model |
| --- | --- |
| 既定の親・build・plan・general | `opencode-go/glm-5.3` |
| explore・light | `opencode-go/glm-5.3-flash` |
| judgment・read-only | `opencode-go/kimi-k3.1-pro` |

画像で利用対象として示された候補からの初期配分であり、性能比較の実測ではない。モデルのallowlist・無効化・自動fallbackは設定しない。OpenCodeとOpenCode Goの両provider経路を残す。Freeを含む手動model切替は可能で、課金・Extra Usage設定は変更しない。

shellと外部directoryへのアクセスは承認確認を残す。固定配置の共通文書・Skillを読むため、その設定directoryへの外部アクセスのみ許可し、同directoryの編集は別に確認する。SessionFolderやAdditional Directoriesの実pathを共通設定へ固定しない。read-onlyとexploreは編集・shell・MCP・再委譲を禁止し、許可された読取系toolだけを使う。planは計画file以外を編集せず、shellを禁止し、read-only/exploreだけへ委譲する。親が必要なコマンド実行・結果保存を担当する。

OpenCodeのpermissionはOS sandboxではない。承認したshellはhostユーザーの権限で動き、文字列・path判定だけで子processのfilesystem/networkアクセスを隔離できない。子は親のpermissionの部分集合ではなく自分の規則を使う。project設定や永続承認も実効値へ影響するため、設定例を管理者policyや強制隔離の代わりにしない。[公式permission仕様](https://opencode.ai/v2/docs/permissions/)

tool許可は、依頼のread-only境界や外部write・破壊的操作・push・protected branch変更等への承認を代替しない。WithMate MCPはtoolごとの承認確認を残し、継続許可の範囲と個別承認が必要な操作をguideと現在のruntime契約で判断する。

## Skillと実行環境

同梱は `session-graph`、`review-test-value`、`design-ui-information`、`relaygraph`、`japanese-tech-writing-review`、`natural-japanese`。日本語文書の2 Skillは既存の明示起動方針を `metadata.opencode/autoinvoke: false` で維持する。Codex専用UI metadata・監査・第三者plugin・外部managed Skillはコピーしない。

| 用途 | 必要環境 |
| --- | --- |
| Session Graph helper | Python 3.10以降 |
| test抽出 | Python 3.11以降、Git |
| TypeScript抽出 | Node.jsとnpm、同梱lockfileの依存 |
| C#抽出 | .NET 8 runtimeと対応SDK、locked restore |
| natural-japanese fixture検証 | Node.js |

TypeScript依存は配置先で導入する。

```powershell
$adapterPath = Join-Path ([Environment]::GetFolderPath('UserProfile')) '.config/opencode/skills/review-test-value/scripts/adapters/typescript'
npm ci --prefix $adapterPath
if ($LASTEXITCODE -ne 0) { throw 'TypeScript adapter dependency installation failed' }
```

SkillはV2のnative discoveryと `skill` toolを使う。AGENTSは標準のinstruction読込を使い、Codex hookやV1 pluginを移植しない。開始・再開・compact後はsession-graphの条件で記録を復元し、子には必要なcontextを明示する。[公式instructions](https://opencode.ai/v2/docs/instructions/) / [skills](https://opencode.ai/v2/docs/skills/)

## WithMate

CLI・Desktopの直接利用とWithMate経由を残す。WithMate対応providerはnative global/project設定を維持し、Character・SessionFolder・Additional Directories等をturn promptへ渡す。Character本文やSession固有値はこのrepositoryへ固定しない。

managed HTTP MCPの `withmate-memory` と `withmate-glossary` はhostがturnごとに供給する接続を使う。URL・短命tokenを実config・設定例へ登録しない。WithMateのSkill同期先にはnative discovery対象の `~/.config/opencode/skills` を指定する。managed `withmate-glossary` はWithMateが配布するため、このrepositoryでは追跡・同名forkしない。

接続・障害調査は[Memory／Character](docs/runbooks/withmate-character-context.md)と[Glossary](docs/runbooks/withmate-repository-glossary.md)、利用判断と継続許可は[WithMate利用方針](docs/guides/withmate.md)を参照する。WithMate本体のOpenCode対応版が必要で、native configの読込成功だけではhostのSession-bound MCPやMemory/Affectの実動作を保証しない。接続確認だけのための書込みや、独自Memory storeへのfallbackは行わない。

## 保守と検証

変更範囲に応じて、設定読込・agent/model/permission・Skill発見と参照、scriptの既存testを確認する。モデルAPIを使わない読込確認と実際のmodel応答・子起動・承認dialog・WithMate接続を分けて報告する。

```powershell
python -X utf8 -m unittest discover -s skills/session-graph/scripts -p "test_*.py"
python -X utf8 -m unittest discover -s skills/review-test-value/scripts -p "test_*.py"
```

Python/TypeScript/C#の変更testは固定task baseから抽出し、全recordをread-only子へ審査委譲する。手順は[変更test](docs/guides/test-changes.md)と同Skillによる。秘密値を含む実設定やservice情報の全文を検証ログへ出さない。

## 課題管理とリリース

本repositoryの不具合・機能要求・後追い残件は[natumekazuki/opencodeのGitHub Issues](https://github.com/natumekazuki/opencode/issues)を正本とする。対応version未指定はバックログとし、Milestoneは使用しない。Issueへの外部writeが未承認なら未登録であることを報告し、SessionFolderのreview記録を登録済みと扱わない。

通常の統合先は `master`。正式版を公開する場合は `vMAJOR.MINOR.PATCH` tag・GitHub Release・`docs/releases/vMAJOR.MINOR.PATCH.md` で対象と確認範囲を示す。task branchのcommitは統合・公開ではない。

維持対象は利用者が使うSkill名・配置入口、各Skillの現行referenceが定める抽出CLI/result/metadata/source adapter/Git選択と、Session Graphの保存・競合・cleanup契約とする。実設定・認証・Memory・既存共有記録等の利用者データはtagの有無にかかわらず保護する。外部modelの提供・費用・品質やWithMate本体は本repositoryの保証に含めない。

内部実装・指示の表現・初期model配分・文書構成そのものは固定しない。維持契約を壊す変更はmajor、互換機能追加はminor、互換修正はpatchとする。tagのない開発途中の実装全体との互換性は保証しないが、データ保護と承認境界は維持する。

実装・review・統合・公開を区別する。必須確認とブロッカー解消、後追いとして採用した残件の登録を満たして統合を判断し、非ブロッカー全件修正は条件にしない。push・masterへのcommit/merge・Issue更新・Release公開は操作/対象/scopeへの明示承認に従う。

## Codex／Claude／OpenCode間の変更反映

この節は `.codex`、`.claude`、OpenCode設定の保守にのみ適用し、共通ルールの適用先repositoryには適用しない。Codexを基本の正本とし、ClaudeとOpenCodeは独立して完結する構成を保つ。

- 相互import・リンク共有には依存しない。
- 共通ルール、自分で保守するSkill、設定例、通知方針は、目的・制約・完了条件を揃えて同じ作業で3者へ反映する。model・tool・path・設定形式等の製品固有部分は各製品へ適応する。
- Claude/OpenCode起点の共通改善はCodexへ逆反映する。製品固有の変更も他2者への影響を確認し、反映しない理由を報告する。
- 認証、実config、Session URL/token、Memory、履歴、cache、外部managed Skillは同期対象外とする。
- 3者の差分と必要な動作を確認し、反映・検証結果を報告する。同期engine・manifest・独自の整合性台帳は置かない。
