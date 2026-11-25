# OpenCode 個人設定

`natumekazuki/opencode` は、OpenCodeに配置する共通指示・Skill・設定例の配布元である。作業時の入口は [AGENTS.md](AGENTS.md)。本書は配布物の導入・検証と、このrepository自身の保守を扱う。他のrepositoryの課題管理先や公開手順を定めるものではない。

## 配置と導入

配置先はユーザーhomeの `~/.config/opencode/`。ここを直接checkoutした端末では、既存差分とbranchを確認して `master` をfast-forwardすると、追跡中の規約とSkillが一緒に更新される。手動配置では次の配布物を**同じrevisionのGit管理ファイルから**選び、相対配置を保つ。端末の設定directoryを丸ごとコピーしない。

| 配布物 | 読み手と用途 |
| --- | --- |
| [AGENTS.md](AGENTS.md) | OpenCodeが読むglobal instruction。projectの指示と組み合わせる |
| [ガイド一式](docs/guides/)、[作業領域](docs/architecture/subagent-workspace.md)、[Glossary接続手順](docs/runbooks/withmate-repository-glossary.md) | 入口・Skillから必要な時に読む。別checkoutやprojectの同名ファイルでは代替しない |
| [skills/](skills/) の同梱6 Skill | 下記のSkill本体と各自のreference・script・license。native discoveryで登録し、本文は `skill` toolで読む |
| [agents/](agents/)、[opencode.example.jsonc](opencode.example.jsonc) | agent定義とモデル・権限の開始値。設定例は実設定とは別物 |
| 導入・診断、記入例 | [用途別runbook](docs/runbooks/)、[ADR・review記録の記入例](templates/)（任意） |

既存の `opencode.json`／`opencode.jsonc` があれば、必要な項目だけを確認して取り込む。今回の規約更新に設定の置換は要らない。新規導入で両方とも存在しない時だけ、設定例を `opencode.jsonc` としてコピーできる。実設定、認証、プラグイン、履歴・cache、WithMateが供給するSkill・接続値・端末固有資料はコピー対象外である。

`opencode debug paths` のconfigと実配置を照合する。globalとprojectの指示は組み合わされ、ガイドのリンク本文まで自動展開されるわけではない。新しいSessionで、使うworkspaceの指示元とSkillの選択元を確認する。[OpenCodeのinstructions](https://opencode.ai/docs/instructions/)・[Skillの探索と優先順位](https://opencode.ai/docs/skills/)

## モデルと権限

設定例の既定は `build`。親・build・plan・generalに `opencode-go/glm-5.3`、explore・lightに `opencode-go/glm-5.3-flash`、judgment・read-onlyに `opencode-go/kimi-k3.1-pro` を指定する。これは手動切替や別の契約経路を制限するallowlistではない。既存Sessionのmodel選択や課金設定は更新しない。

設定例はshellと外部directoryを原則askとし、共通設定directoryへのアクセスはallow、編集はaskとする。planは計画file以外の編集・shellをdenyする。OpenCode nativeの `subagent` はpolicyで拒否し、WithMateの通常子は[別経路](docs/architecture/subagent-workspace.md#nativeサブエージェント)で扱う。agentの存在はその呼出しの許可ではない。

実効値はglobal・project・hostの設定で決まる。permissionはOS sandboxではなく、許可したshellはhostユーザーとして動く。toolが使えても、操作の承認・依頼の範囲・上位policyは変わらない。[V2の設定](https://opencode.ai/docs/config/)・[権限](https://opencode.ai/docs/permissions/)

## SkillとWithMate

- [design-ui-information](skills/design-ui-information/SKILL.md)、[review-test-value](skills/review-test-value/SKILL.md)、[relaygraph](skills/relaygraph/SKILL.md)、[session-graph](skills/session-graph/SKILL.md) は各descriptionの条件で使う。
- [japanese-tech-writing-review](skills/japanese-tech-writing-review/SKILL.md) と [natural-japanese](skills/natural-japanese/SKILL.md) は明示起動用。`metadata.opencode/autoinvoke: false` を保つ。
- 変更testの抽出にはGit、Python 3.11以降、TypeScriptにはNode.js・npm、C#には.NET 8 runtimeと対応SDK・locked restoreが必要。TypeScriptの依存は配布した `skills/review-test-value/scripts/adapters/typescript` で `npm ci` を実行する。

WithMateのGlossary／Session Graph／SessionのHTTP接続とmanaged `withmate-glossary` SkillはWithMateが供給する。通常の導入で手動登録・forkや短命tokenの永続化はしない。Glossaryの手動STDIOはrunbookの開発・診断用途に限る。通常子の選定・配送は[協同guide](docs/guides/withmate-collaboration.md)、Worktreeの配置先決定と実体の確認は[作業領域guide](docs/architecture/subagent-workspace.md#git-worktreeの配置)に従う。端末固有の方針ファイルは配布・自動移行しない。

## 配布時の確認

変更が影響する契約を確認する。配布元の文書を確認しただけなら、端末での適用や実動作まで確認したとは扱わない。出力比較が依頼された場合は、[比較手順](docs/runbooks/compare-subagent-roles.md#役割指示を比較する時)を参照する。

モデル実行やsecretを公開CIへ追加しない。確認した対象・条件・結果と判断に響く未確認は既存の[review記録](docs/guides/development.md#sessionfolderのreview記録と対応管理)等に残し、文書の一致を実動作の証拠にしない。

配置は `opencode debug paths`、設定元・agentは `opencode debug config`／`debug agents`、Skillは `opencode api skill.list` で確認できる。出力には実設定やSkill本文も含まれるので、ログには対象のpath・ID・必要な判定だけを出す。Global instructionとSkillは別の読込経路であり、ファイルの存在、実際の読込、生成応答、WithMateでの実配送を区別する。協同経路を変更した時は、実際の送信元・宛先と保存・再開を対応させる。

抽出器とadapterの既存回帰テスト：

```powershell
python -X utf8 -m unittest discover -s skills/review-test-value/scripts -p "test_*.py"
```

変更したtestの独立審査は[変更testガイド](docs/guides/test-changes.md)に従う。

## このrepositoryの管理先とリリース境界

バグ・後追い残件の管理先は [natumekazuki/opencode のGitHub Issues](https://github.com/natumekazuki/opencode/issues)。Issueの登録・更新・closeは外部writeとして、共通入口の承認範囲に従う。

### 本repositoryの開発・リリース境界

- `vMAJOR.MINOR.PATCH` タグが指すcommitを配布版とする。タグのないcommitやbranch上の途中状態は互換保証の対象としない。
- 維持するのは各タグのリリースノートで明示した利用者向け契約であり、実装全体ではない。[初回の維持対象 v1.0.0](docs/releases/v1.0.0.md) から `review-test-value` のCLI、構造化result、metadata v2、対応adapter、Git差分選択を含む。詳細は[契約別の参照文書](skills/review-test-value/references/)で確認する。
- 破壊的な契約変更はmajor、互換な変更はminorまたはpatchを上げて新しいタグへ記録する。リリースごとに [一覧](docs/releases/) と `docs/releases/vMAJOR.MINOR.PATCH.md` を更新し、変更・互換性・検証結果を残す。

統合先は `master`。GitHub Releaseと版別ノートは同じタグで公開する。今回の変更は [v3.1.2](docs/releases/v3.1.2.md) を参照。Skill名・導入と配置の入口・Session GraphのMCP利用方針も維持対象である。MCPのschema・認可はruntimeが定める。

### Codex／Claude／OpenCode間の変更反映

共通ルール・自主管理Skill・設定例・通知方針はCodexを基本の正本とし、通常の保守では目的・制約・完了条件を揃えて同じ作業で3者に反映する。各配布元（`.codex`、`.claude`、`~/.config/opencode`）は相互importや共有リンクに依存せず独立して使える構成とする。Claude・OpenCode起点の共通改善はCodexにも戻す。

model・tool・設定形式など製品固有の機構は各製品に合わせる。対象製品を限定した依頼を他の配布元への書込権限としない。認証、実config、Session URL・token、Memory・履歴・cache、外部managed Skillは同期しない。
