# OpenCode 個人設定

`natumekazuki/opencode` は、OpenCodeに配置する共通指示・Skill・設定例の配布元である。作業時の入口は [AGENTS.md](AGENTS.md)。本書は配布物の導入・検証と、このrepository自身の保守を扱う。他のrepositoryの課題管理先や公開手順を定めるものではない。

## 構成

| 探したいもの | 場所 |
| --- | --- |
| 作業中の判断・必須参照 | [共通入口](AGENTS.md)、[用途別guide](docs/guides/)、[委譲と作業領域](docs/architecture/subagent-workspace.md) |
| 専門手順・実行資産 | [配布するSkill](skills/)、[native role](agents/) |
| 親・子の設定 | [共有config例](opencode.example.jsonc) |
| 導入・診断、記入例 | [用途別runbook](docs/runbooks/)、[ADR・review記録の記入例](templates/)（任意） |
| 過去の判断と公開版 | [ADR](docs/adr/)、[版別リリースノート](docs/releases/README.md)（変更に伴う影響） |

## このrepositoryの管理先とリリース境界

バグ・後追い残件の管理先は [natumekazuki/opencode のGitHub Issues](https://github.com/natumekazuki/opencode/issues)。Issueの登録・更新・closeは外部writeとして、共通入口の承認範囲に従う。

### 本repositoryの開発・リリース境界

- `vMAJOR.MINOR.PATCH` タグが指すcommitを配布版とする。タグのないcommitやbranch上の途中状態は互換保証の対象としない。
- 維持するのは各タグのリリースノートで明示した利用者向け契約であり、実装全体ではない。
- 破壊的な契約変更はmajor、互換な変更はminorまたはpatchを上げて新しいタグへ記録する。リリースごとに [一覧](docs/releases/README.md) と `docs/releases/vMAJOR.MINOR.PATCH.md` を更新し、変更内容とその互換性・移行・利用・安全上の影響を記録する。

統合先は `master`。GitHub Releaseと版別ノートは同じタグで公開する。公開版の変更は[版別ノート](docs/releases/README.md)を参照する。Skill名・導入と配置の入口・Session GraphのMCP利用方針も維持対象である。MCPのschema・認可はruntimeが定める。

### この設定ホームでGit管理するもの

新しいパスは既定で除外し、配布専用の文書・開発資産と、明示した設定例・ソースだけを許可する。Skillはパッケージごとに登録する。実設定・認証、端末専用のSkill／pluginや生成物は配布しない。Gitignoreは追跡済みファイルの変更や機密の中身までは除外しないので、stageする内容も確認する。

新しい配布物を追加するときは、必要な親ディレクトリと対象だけを `.gitignore` で許可し、`git check-ignore -v --no-index -- <path>` と `git add -n -- <path>` で正しく追加できるか確かめる。除外を変更するときは、Git Bash等で `git ls-files -z | git check-ignore -z --no-index --stdin` を実行し、既存の追跡パスが出力されないことも確認する。`git add -f` で回避せず、実設定とは別の見本を配布する。

### Codex／Claude／OpenCode間の変更反映

共通ルール・自主管理Skill・設定例・通知方針はCodexを基本の正本とし、通常の保守では目的・制約・完了条件を揃えて同じ作業で3者に反映する。各配布元（`.codex`、`.claude`、`~/.config/opencode`）は相互importや共有リンクに依存せず独立して使える構成とする。Claude・OpenCode起点の共通改善はCodexにも戻す。

model・tool・設定形式など製品固有の機構は各製品に合わせる。対象製品を限定した依頼を他の配布元への書込権限としない。認証、実config、Session URL・token、Memory・履歴・cache、外部managed Skillは同期しない。

## 配置と検証

共通文書の読取先は実行ユーザーのhome直下の `~/.config/opencode/`。直接checkoutするか、別checkoutから次の配布物を同じ相対配置にコピー・linkする。配布元を編集しても、別の端末配置は更新されない。

- `AGENTS.md`、`docs/guides/` 全体、`docs/architecture/subagent-workspace.md`、[Glossaryの接続・障害調査runbook](docs/runbooks/withmate-repository-glossary.md) を揃える。別repositoryのrootと下位ディレクトリから、実際に読んだ共通入口・詳細と、そのrepository固有の管理先・互換性定義を確認する。参照が欠けたら配置を修復し、cwdや別の `XDG_CONFIG_HOME` の同名文書で埋めない。

- WithMateのGlossary／Session Graph／SessionのHTTP接続とmanaged `withmate-glossary` SkillはWithMateが供給する。通常の導入で手動登録・forkや短命tokenの永続化はしない。Glossaryの手動STDIOはrunbookの開発・診断用途に限る。通常子の選定・配送は[協同guide](docs/guides/withmate-collaboration.md)、Worktreeの配置先決定と実体の確認は[作業領域guide](docs/architecture/subagent-workspace.md#git-worktreeの配置)に従う。端末固有の方針ファイルは配布・自動移行しない。

### 配置と導入

配置先はユーザーhomeの `~/.config/opencode/`。ここを直接checkoutした端末では、既存差分とbranchを確認して `master` をfast-forwardすると、追跡中の規約とSkillが一緒に更新される。手動配置では次の配布物を**同じrevisionのGit管理ファイルから**選び、相対配置を保つ。端末の設定directoryを丸ごとコピーしない。

既存の `opencode.json`／`opencode.jsonc` があれば、必要な項目だけを確認して取り込む。今回の規約更新に設定の置換は要らない。新規導入で両方とも存在しない時だけ、設定例を `opencode.jsonc` としてコピーできる。実設定、認証、プラグイン、履歴・cache、WithMateが供給するSkill・接続値・端末固有資料はコピー対象外である。

`opencode debug paths` のconfigと実配置を照合する。globalとprojectの指示は組み合わされ、ガイドのリンク本文まで自動展開されるわけではない。新しいSessionで、使うworkspaceの指示元とSkillの選択元を確認する。[OpenCodeのinstructions](https://opencode.ai/docs/instructions/)・[Skillの探索と優先順位](https://opencode.ai/docs/skills/)

## modelとrole

設定例の既定は `build`。親・build・plan・generalに `opencode-go/glm-5.3`、explore・lightに `opencode-go/glm-5.3-flash`、judgment・read-onlyに `opencode-go/kimi-k3.1-pro` を指定する。これは手動切替や別の契約経路を制限するallowlistではない。既存Sessionのmodel選択や課金設定は更新しない。

設定例はshellと外部directoryを原則askとし、共通設定directoryへのアクセスはallow、編集はaskとする。planは計画file以外の編集・shellをdenyする。OpenCode nativeの `subagent` はpolicyで拒否し、WithMateの通常子は[別経路](docs/architecture/subagent-workspace.md#nativeサブエージェント)で扱う。agentの存在はその呼出しの許可ではない。

実効値はglobal・project・hostの設定で決まる。permissionはOS sandboxではなく、許可したshellはhostユーザーとして動く。toolが使えても、操作の承認・依頼の範囲・上位policyは変わらない。[V2の設定](https://opencode.ai/docs/config/)・[権限](https://opencode.ai/docs/permissions/)

## Skillの配布

このrepositoryで保守するのは、UIの [design-ui-information](skills/design-ui-information/)、Crewの共有記録を扱う [session-graph](skills/session-graph/)、採用済みrepositoryを扱う [relaygraph](skills/relaygraph/)、明示的な文章推敲用の [japanese-tech-writing-review](skills/japanese-tech-writing-review/) と [natural-japanese](skills/natural-japanese/) である。導入時はSkill本文とmetadataを揃える。

`natural-japanese` の[同期元](https://github.com/coji/natural-japanese)・commit・licenseは [NOTICE](skills/natural-japanese/NOTICE) と [LICENSE](skills/natural-japanese/LICENSE) を参照。

- [japanese-tech-writing-review](skills/japanese-tech-writing-review/SKILL.md) と [natural-japanese](skills/natural-japanese/SKILL.md) は明示起動用。`metadata.opencode/autoinvoke: false` を保つ。

## このrepositoryのGit pre-commit

この配布元のcheckoutで、Python 3.12を使う。有効にする前に `git config --show-origin --get core.hooksPath` で既存の設定を確認し、別のhookがある場合は上書きせず併用方法を決める。このrepositoryの既存設定と競合しなければ `git config --local core.hooksPath .githooks` で有効化できる。同じGit repositoryのworktreeにも影響する。設定を変えずに一度だけ使う場合は `git -c core.hooksPath=.githooks commit`、事前に同じ検査を手動で行う場合は `python -B -X utf8 scripts/check-repo.py --staged` を使う。

staged差分の空白・競合マーカーと、このrepositoryの検査スクリプトで追加・変更したPython宣言のdocstringを検査する。外部由来の `natural-japanese` はdocstring検査に含めず、関連する変更ではPythonの構文と同梱fixtureによる実行例を確認する。設定例を変更した場合は、そのJSON構文を確認する。本文コメントの意味はreviewで確認する。CIは変更差分を検査し、Windows・Ubuntuで維持対象の検査を実行する。Git hookはOpenCodeの実行時設定とは別で、チェックアウトしただけでは有効にならない。

## 配布時の確認

変更が影響する契約を確認する。配布元の文書を確認しただけなら、端末での適用や実動作まで確認したとは扱わない。出力比較が依頼された場合は、[比較手順](docs/runbooks/compare-subagent-roles.md#役割指示を比較する時)を参照する。

モデル実行やsecretを公開CIへ追加しない。確認した対象・条件・結果と判断に響く未確認は既存の[review記録](docs/guides/development.md#sessionfolderのreview記録と対応管理)等に残し、文書の一致を実動作の証拠にしない。

配置は `opencode debug paths`、設定元・agentは `opencode debug config`／`debug agents`、Skillは `opencode api skill.list` で確認できる。出力には実設定やSkill本文も含まれるので、ログには対象のpath・ID・必要な判定だけを出す。Global instructionとSkillは別の読込経路であり、ファイルの存在、実際の読込、生成応答、WithMateでの実配送を区別する。協同経路を変更した時は、実際の送信元・宛先と保存・再開を対応させる。
