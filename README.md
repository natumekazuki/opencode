# OpenCode 個人設定

OpenCode V2で使う共通指示・Skill・agent・設定例の配布元。毎回読む入口は [AGENTS.md](AGENTS.md)、詳しい判断はそこから指定するガイドに置く。このREADMEは配布物の導入・保守に使う。

## 配置と導入

配置先はユーザーhomeの `~/.config/opencode/`。ここを直接checkoutした端末では、既存差分とbranchを確認して `master` をfast-forwardすると、追跡中の規約とSkillが一緒に更新される。手動配置では次の配布物を**同じrevisionのGit管理ファイルから**選び、相対配置を保つ。端末の設定directoryを丸ごとコピーしない。

| 配布物 | 読み手と用途 |
| --- | --- |
| [AGENTS.md](AGENTS.md) | OpenCodeが読むglobal instruction。projectの指示と組み合わせる |
| [ガイド一式](docs/guides/)、[作業領域](docs/architecture/subagent-workspace.md)、[Glossary接続手順](docs/runbooks/withmate-repository-glossary.md) | 入口・Skillから必要な時に読む。別checkoutやprojectの同名ファイルでは代替しない |
| [skills/](skills/) の同梱6 Skill | 下記のSkill本体と各自のreference・script・license。native discoveryで登録し、本文は `skill` toolで読む |
| [agents/](agents/)、[opencode.example.jsonc](opencode.example.jsonc) | agent定義とモデル・権限の開始値。設定例は実設定とは別物 |

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

WithMateはCharacter・SessionFolder・Additional DirectoriesをTurnごとに渡し、Glossary・Session Graph・Sessionのmanaged HTTP MCPを当該Sessionへ供給する。同期先はOpenCodeの探索対象とし、managed `withmate-glossary` の同名forkや短命tokenの手動登録はしない。接続の診断は[runbook](docs/runbooks/withmate-repository-glossary.md)、通常子の実行条件・相談・配送は[協同ガイド](docs/guides/withmate-collaboration.md)に従う。

## 保守と検証

比較が依頼された結果に必要な場合は、判断に効く条件を選び、条件差が結論を左右するならその範囲を示す。条文や文字数だけから、生成応答の改善を結論づけない。

配置は `opencode debug paths`、設定元・agentは `opencode debug config`／`debug agents`、Skillは `opencode api skill.list` で確認できる。出力には実設定やSkill本文も含まれるので、ログには対象のpath・ID・必要な判定だけを出す。Global instructionとSkillは別の読込経路であり、ファイルの存在、実際の読込、生成応答、WithMateでの実配送を区別する。協同経路を変更した時は、実際の送信元・宛先と保存・再開を対応させる。

抽出器とadapterの既存回帰テスト：

```powershell
python -X utf8 -m unittest discover -s skills/review-test-value/scripts -p "test_*.py"
```

変更したtestの独立審査は[変更testガイド](docs/guides/test-changes.md)に従う。検証結果は[review記録](docs/guides/development.md#sessionfolderのreview記録と対応管理)等に残す。

## 課題管理とリリース

本リポの不具合・機能要求・後追いは[GitHub Issues](https://github.com/natumekazuki/opencode/issues)で管理し、対応版未指定はバックログとする。統合先は `master`。統合・外部登録・公開は別の操作であり、[開発ガイド](docs/guides/development.md)の条件と操作権限に従う。

維持対象はSkill名・配置入口、変更test抽出器のCLI・result・metadata・source adapter・Git選択契約、Session Graph SkillのMCP利用方針である。MCPのschemaと認可はruntimeが定める。既存データと実設定はtagの有無によらず保護する。内部実装・指示の表現・初期model配分は固定せず、外部modelの提供・費用・品質やWithMate本体は保証対象としない。

維持契約の破壊はmajor、互換機能追加はminor、互換修正はpatchとする。公開に先立ち[版別リリースノート](docs/releases/)と案内を更新する。今回の変更と更新方法は [v3.1.0](docs/releases/v3.1.0.md) を参照。正式なtag・GitHub Releaseの公開は統合とは別に行う。

## 配布元間の保守

共通方針の基準はCodexの配布元とし、OpenCodeとClaudeの配布元は各製品で独立して利用できるように保つ。共通する変更の目的・制約・完了条件を照合し、OpenCode起点の改善も共通側へ返す。起動・tool・model・権限・設定形式は各製品の実際の経路に合わせる。これは配布元の保守方針であり、他製品への実行時参照や依頼外の書込み権限ではない。
