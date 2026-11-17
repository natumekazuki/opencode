# 委譲と作業領域

## WithMateの通常子Sessionとnativeサブエージェント

OpenCodeのnative `subagent` toolによる呼び出しは無効とする。WithMate通常子Sessionの選択・依頼と、統括が直接行える範囲は[協同guide](../guides/withmate-collaboration.md#担当の選択と依頼)に従う。必須の独立審査は免除せず、利用可能な独立担当がいなければ完了判断を保留する。

WithMateの子は別の参加者であり、[Session間の協同](../guides/withmate-collaboration.md)でRole・背景確認・Coordination・明示報告・Turn終了と再開を扱う。同じ成果の続き・修正・受入対応には継続可能な既存Sessionを使い、新規との判断も同guideに従う。

通常のMain Sessionは役割を問わず、全体統括・個別統括・Workerを含めて1 Session・1専有Workspaceとし、同じ作業directoryを複数のSessionへ重複割当しない。他SessionのWorkspaceを追加directory等で直接編集して分離を迂回せず、許可されたread-only参照と共同編集は区別する。Auxiliaryとnativeサブエージェントはこの必須ルールの対象外であり、既存の編集所有・衝突回避・権限はそのまま保つ。

WithMate通常子は既存の委譲許可、必須専門審査、操作権限とruntime制限に従い、[実行条件の選定](../guides/withmate-collaboration.md#実行条件の選定)は協同guideに従う。

## nativeサブエージェント

配布用設定例は `tool.use` policyで `subagent:*` をdenyする。これはOpenCode nativeの呼び出しに適用し、WithMate通常子Sessionの操作を禁止しない。設定のmergeや上位policyの影響は実効設定で確認する。

## 依頼と結果

reviewでは主要動作、後続の依存、[開発・review・統合](../guides/development.md)の統合基準を渡す。共有判断に関わる場合は採用済み方針・適用状況・未合意事項と[設計判断・文書化](../guides/design-decisions.md)も渡す。調査・草案作成の委譲を方針採用や操作権限の拡張と解釈しない。

test価値reviewは[review-test-value](../../skills/review-test-value/SKILL.md)に従い、全recordを独立したread-only担当へ渡す。親が必要なtest実行・diff抽出・結果保存を行い、担当は許可されたtoolでproduction/fixture/oracleを確認する。必要な情報がtool制約で取得できなければ、親へ不足を返す。

担当fileと生成物の所有を明示し、他者が並行作業中であること、他者の編集を巻き戻さないことを伝える。結果は根拠・確認範囲・未解決を親へ返し、専用の管理基盤や結果台帳を新設しない。親は指摘の採否・対応時期・統合判断を行い、必要な結果記録をSessionFolderへ保存する。

## 子の作業領域と親の準備

子を起動する直接の親が、起動前にその子の専用作業領域を作成・確認して絶対pathを渡す。全体統括は自身が起動する個別統括とWorkerの分を、個別統括は自身のWorkerの分を準備する。準備に失敗したら子の起動を保留し、親のWorkspaceを代用しない。子を親のWorkspaceで起動して自己準備させず、領域の準備・回収だけを目的にWorkerを起動しない。Worktree準備だけを直接依頼された場合も、受けた統括が自身で実施する。作業領域の管理は統括の直接操作に含めるが、短い実作業全般を統括へ戻す例外にはしない。

Git管理下の通常子には専用Worktreeを割り当てる。同じrepositoryでの並行作業は別のbranchで行う。Git管理外の作業は[非Gitの作業領域](#非gitの作業領域)に従う。配置先の取得と結果の使い分けは、協同guideの[Worktreeの配置先](../guides/withmate-collaboration.md#worktreeの配置先)を正本とする。

## Git worktreeの配置

commit固定は具体的な隔離・再現の必要に応じて使い、全reviewへ一律には要求しない。Git管理下の通常子Sessionには、reviewを含めて専用Worktreeを割り当てる。Auxiliaryとnativeサブエージェントでは、具体的な隔離・再現の必要に応じてWorktreeを使う。非Gitの場合は[非Gitの作業領域](#非gitの作業領域)に従う。ここでは独自のroot・分類・layoutやpath組み立てを定義しない。

親が、登録先の実体とbranch、分岐元のbranch・commit、作成先の占有、未コミット変更を確認し、必要なGit操作を行う。返された既存Worktreeは他Sessionの占有を確認してから割り当て、無条件に共有しない。forceで同じbranchを複数checkoutしない。配置候補の取得を作成済み・利用可能の証拠とせず、準備した実体の絶対pathを子のWorkspaceへ渡す。既存worktreeの移設・削除や他者の作業領域の上書きは、この準備の許可に含めない。

## 作業領域の再利用と回収

子の作業領域は同じSession・同じ担当作業で再利用し、Turn単位で作り直さない。待機・中断・修正待ちの間は保持し、idleや経過日数だけで削除しない。親は子の成果を受入し、必要な統合の後に回収する。回収前に、実行中や予約済みの作業と他Sessionの利用がないこと、成果と資料の保全、未コミット・未追跡・ignoredファイルに残すべきデータがないことを確認する。Worktreeとbranchは別々に回収できる。未統合のcommitや採否未決の成果を無断で破棄せず、中止・不採用の場合は保存か破棄かを決めてから扱う。通常の削除が拒否されたら、forceへの自動的な切り替えはしない。

親が回収できるのは、この運用で新規に作った子用のWorktree、ローカルtask branch、非Git専用領域に限る。無関係な既存領域とremote・default・protected branchは対象外とする。回収の管理台帳は新設せず、再開時は保存されたbranch・commitと保全した成果から親が作業領域を再準備する。

子Sessionへの送信が割当Workspaceの不在を理由に拒否された場合、送信元Sessionが復旧を担う。Git管理下では保存したbranch・commitから元の割当pathへWorktreeを再準備し、非Gitでは保全した入力から専用領域を再作成する。いずれも利用可能な実体を確認してから改めて送信する。拒否時に未配送（`effect: none`）が確認できた場合に限って再送し、受付結果が不明なら同じrequestIdと宛先で状態を確認して重複送信を避ける。送信元が安全に復旧できない場合は再送を保留して報告し、子Sessionに復旧を委ねない。

## 非Gitの作業領域

Git管理外の作業では、親が専用directoryへ今回必要な入力だけをコピーし、子は原本を直接編集しない。親は原本の現在状態と対象を確認して、必要な成果だけを反映する。directory全体の上書き戻しはしない。継続した開発や並行統合が見込まれる場合はGitでの管理を提案し、合意がない限りgit initや全ファイルのcommitをしない。DBや外部サービス等、directoryの分離だけでは隔離できない対象は、準備の前に方式を相談する。非Git専用領域の回収も、この運用で新規に作った専用領域だけを対象に、[作業領域の再利用と回収](#作業領域の再利用と回収)と同じ条件で行う。
