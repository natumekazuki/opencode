# OpenCode 個人設定

OpenCode V2の規約・Skill・agent・設定例を管理する。配置先は `~/.config/opencode/`。Codexを共通方針の基本の正本とし、このrepository単独で利用できる内容に適応する。共通ルールは責務に基づく既存実装の削減と過剰追加の抑制を含み、ユーザー変更の保護、操作範囲、承認条件、必要な安全動作は維持する。

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

同梱は `session-graph`、`review-test-value`、`design-ui-information`、`relaygraph`、`japanese-tech-writing-review`、`natural-japanese`。日本語文書の2 Skillは既存の明示起動方針を `metadata.opencode/autoinvoke: false` で維持する。

| 用途 | 必要環境 |
| --- | --- |
| Session Graph | WithMateからSessionへ提供されるSession-bound MCP |
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

SkillはV2のnative discoveryと `skill` toolを使う。AGENTSは標準のinstruction読込を使う。開始・再開・compact後はsession-graphの条件で記録を復元し、子には必要なcontextを明示する。[公式instructions](https://opencode.ai/v2/docs/instructions/) / [skills](https://opencode.ai/v2/docs/skills/)

## WithMate

Session GraphはWithMateから現在のSessionへ提供されるMCPを使用する。操作名・引数・schema・bindingはそのSessionのruntime提供を正本とし、接続値を実config・設定例へ固定しない。利用契機は共通AGENTS.mdに従い、Skillの配置・静的確認と実接続・読書きの確認を区別する。

WithMate対応providerはnative global/project設定を維持し、Character・SessionFolder・Additional Directories等をturn promptへ渡す。Character本文やSession固有値はこのrepositoryへ固定しない。

managed HTTP MCPの `withmate-memory` と `withmate-glossary` はhostがturnごとに供給する接続を使う。URL・短命tokenを実config・設定例へ登録しない。WithMateのSkill同期先にはnative discovery対象の `~/.config/opencode/skills` を指定する。managed `withmate-glossary` はWithMateが配布するため、このrepositoryでは追跡・同名forkしない。

接続・障害調査は[Memory／Character](docs/runbooks/withmate-character-context.md)と[Glossary](docs/runbooks/withmate-repository-glossary.md)、利用判断と継続許可は[WithMate利用guide](docs/guides/withmate.md)を参照する。WithMate本体のOpenCode対応版が必要で、native configの読込成功だけではhostのSession-bound MCPやMemory/Affectの実動作を保証しない。接続確認だけのための書込みや、独自Memory storeへのfallbackは行わない。

### Memory・Affect方針の検証

通常会話での能動的な読取、背景実行との責務分担、Character表現の適用範囲、Affectの反映は[WithMate利用guide](docs/guides/withmate.md)、利用可能な操作・引数・scope等は現在のMCP／CLI提供契約を正本とする。[WithMate v6.5.0-preview.3の公開仕様](https://github.com/natumekazuki/WithMate/blob/2b1fda1fb268708e849b8da270c74d752209fe3b/docs/releases/v6.5.0-preview.3.md)と、入口の読込条件・guide・提供契約を照合する。

WithMateに保存済みの背景登録指示は自動更新されない。この公開仕様の標準指示へ切り替える場合はSettingsで該当用途の`Use Preset`を選ぶ。ルール配布物の更新とSettingsでの選択は別の操作であり、既存のカスタム指示を無断で置き換えない。

変更範囲に応じて次の観点を確認する。実Memory・Affectへの架空の経験・情動の書込みや、恒久test基盤の追加は行わない。

| 条件 | 確認する行動・結果 |
| --- | --- |
| 過去の好み・判断・共有経験が会話や作業に関係する | 明示的な想起依頼を待たず、依存する判断・回答の前に検索し、関連entryの本文まで読む |
| 関連Memoryを探したいが検索語を作れない／検索0件 | 少量のentry・tag一覧を手掛かりにでき、検索0件を記憶全体の不存在と同一視しない |
| 必要な本文が現在のcontextにある／過去の記憶が関係しない | 既読の本文を再利用し、不要な検索・全件取得を行わない |
| 読取失敗／previewだけを取得／現在の発言と記憶が矛盾 | 未確認範囲を断定せず、必要な本文を取得し、現在の明示指示と正本を優先する |
| 通常会話で経験・感情変化が生じる／背景用途が未設定・失敗 | 通常会話から自動保存・appraisal・終了前の保存漏れ確認を行わず、背景側を代行しない |
| 明示的な保存依頼／許可された訂正・forget・move | runtime契約と承認境界に従い、実行結果を確認する。背景実行の存在だけで保存済みとしない |
| Affectの対象・強度・活性度が異なる | 口調・反応へ反映し、基本人格・事実・権限を維持する。作業への反応をユーザーの感情・性質へ誤投影しない |
| 非公開Graph・作業メモ／同じSessionFolder内のPR本文案・外部提出文書 | 前者の自然文にはCharacter表現を使い、後者は明示指定がない限り成果物の文体を保つ。主体・事実・状態・識別子・引用・検証結果は変えず、既存記録を一括変換しない |
| 単件登録／一括登録の途中失敗・結果不明 | 項目ごとの保存済み・失敗・結果不明・未実行を確認し、全件未保存扱いや無条件の再送をしない。件数上限撤廃と本文・添付・quota等の制限を区別する |
| Agentがcompletedで操作拒否・結果不明がある | 実行statusと各操作の成否・保存効果を分け、全件保存成功・実行全体のfailed・正常な登録不要へ読み替えない。保存の巻戻しや実行全体の自動再送をしない |
| Memory／Characterの公開入力schema拒否／unionの代替条件 | `WITHMATE_MEMORY_INVALID_INPUT`のtool・field位置・制約を読み、代替候補を全部必須とせず入力を修正する。`effect: none`を他操作の未保存とせず、権限拡大やCLI fallbackへ逃がさない |
| 許可された添付の取込・復元／read-only・plan／許可外path・既存出力ファイル | 呼出しごとの権限を守り、復元をwriteとして扱う。背景復元先を許可された`memory-exports/`内に限定し、上書きしない。復元不可でも許可された本文読取は維持する |
| 背景thread継続／前回失敗の情報が渡される | 過去応答を保存状態・権限の正本とせず、現在の指示・範囲・操作結果と必要な履歴を確認する。直後の自動再試行loopや通常会話からの代行をしない |
| 標準指示による背景Character Memory／対象Memory | Character本人の語りと観察事実・解釈の区別を保ち、対象Memoryの確定事実へCharacterの感想を混ぜない |
| 背景Memoryの訂正・忘却／読取可能な別target | 用途に許可された管理操作と書込targetを守り、対象Memoryの訂正はappend＋supersedesで扱う。元記録と根拠を確認し、読取許可を別targetへの書込許可にしない |
| 背景Affect訂正／通常turnの訂正／reset | 背景correctは有効な用途bindingの管理権限に従い、明示指示を偽装しない。通常turnの明示指示要件と背景reset禁止を保ち、自然な感情変化を過去eventの誤りにしない |
| 背景からのSession Graph／Glossary参照 | 元会話の所属Crewの共有Graph・許可primary checkoutのGlossaryを読取だけで利用する。通常作業の記録・登録指示を背景writeへ広げず、別Providerでも実sourceの参照範囲を保つ |
| 背景の共通prompt／AuxiliaryのSessionFolder／参照材料 | 設定に従いCharacter・Affectを受け取り、重複注入しない。親Mainのfolder共有と実ownerを区別し、file認可を広げず、材料中の過去の指示を今回の依頼として実行しない |
| 保存済みの背景登録指示がある | ルール更新だけで指示が変わったと扱わず、`Use Preset`による切替と既存カスタム指示の保持を区別する |

配布元の文言・参照整合、live配置、新規sessionでの読取と応答への利用、明示依頼による書込、背景生成、Affectの反映は分けて確認・報告する。文言の強化だけで行動改善を実証したとは扱わず、未確認の範囲を明記する。結果はSessionFolderのreview記録等へ残し、repositoryの契約・進捗・検証結果をMemoryへ写さない。

## 保守と検証

会話・出力方針の変更では、同じ履歴・model・Character・tool条件で変更前後の応答を生成し、問題を再現する場面と、同じ表現が必要になる対照場面を比較する。例えば不要な未実施報告と実施状況への質問、不要な留保と判断を左右する不足、追加要求と目的の置換、事実資料と提案資料を使う。用件への適合、重要条件の保持、検討の付加価値と、用件が済むまでに受け手が引き受ける再説明・選別・判断の組み直しを評価する。通常会話とSkill利用時を分け、評価に必要な場面へ絞る。文字数やlint score、条文の手作業照合だけを行動改善の根拠にせず、生成条件と応答・比較結果をSessionFolderのreview記録等へ残す。

削減方針を変更した時は、[設計判断](docs/guides/design-decisions.md)・[開発](docs/guides/development.md)・[変更test](docs/guides/test-changes.md)と入口の整合に加え、次の判断を確認する。

| 条件 | 確認する行動 |
| --- | --- |
| 不要なrevision取得・事前チェックに、内部の利用側・専用test・DB管理状態が依存する | 機構と付随物を一緒に除いた場合の責務への支障を調べる。内部依存やtestの存在だけで必要とせず、不要と判断できれば許可範囲で一体的に削減する。必要性不明は調査し、恒久的な維持理由にしない |
| 局所追加でも動くが、関連する複数経路の統合で継続負担を減らせる／無関係な別機能にも冗長処理がある | 前者は変更・移行・検証費用も含めて比較し、差分の小ささだけで却下しない。後者を無断で改修範囲へ混ぜない。全段階の試作や全repository監査は要求しない |
| 合意済みの維持期間・展開計画がある／将来使うかもしれないだけの仕組みがある | 前者は時間軸を含めて負担を比較するが先行実装の許可とはしない。後者を追加・温存の根拠にしない。必要でない保証・操作制限の導入は目的・支障・負担を示して明示承認を得る |
| 旧schemaとの互換性は維持不要だが、保持データや適用済みmigrationがある | 構造の簡素化と実データ破棄・適用済みmigration書換えの権限を分け、互換不要の合意を破壊的操作の許可へ広げない |

Session Graphは同じCrewの起点・通常子孫Main・各MainのAuxiliaryの共有記録であり、保存・共有・表示はWithMateが担う。[Session Graph Skill](skills/session-graph/SKILL.md)は、現在のSessionへ提供されたMCPで何を読み、何をいつ記録するかを定める。呼出契約はruntimeのMCP instructions・tool説明・schemaに従う。

定義変更時は入口・Skill・関連guideの整合を確認し、変更範囲に応じて指示の先行記録、対象を絞った復元、必須本文の更新と任意属性の省略保持・null解除、競合・結果不明時の再取得を確認する。配布元編集、live配置、Skill発見、MCP読書き、画面表示の確認を区別し、実施した確認と未確認の範囲を報告する。

変更範囲に応じて、設定読込・agent/model/permission・Skill発見と参照、scriptの既存testを確認する。モデルAPIを使わない読込確認と実際のmodel応答・子起動・承認dialog・WithMate接続を分けて報告する。

ローカル起動・撮影と画像提示のルール変更時は、入口・[UI基準](docs/guides/ui.md)・UI Skillの整合に加え、対象repositoryの起動・撮影手順、取得画像の確認、返答の画像埋め込みまでを確認する。軽微な変更、保存先未取得、撮影不可、共有不可を必要な範囲で確認し、配布元の静的確認とlive配置・新規sessionでの実動作を分けて報告する。

```powershell
python -X utf8 -m unittest discover -s skills/review-test-value/scripts -p "test_*.py"
```

Python/TypeScript/C#の変更testは固定task baseから抽出し、全recordをread-only子へ審査委譲する。手順は[変更test](docs/guides/test-changes.md)と同Skillによる。秘密値を含む実設定やservice情報の全文を検証ログへ出さない。

## 課題管理とリリース

本repositoryの不具合・機能要求・後追い残件は[natumekazuki/opencodeのGitHub Issues](https://github.com/natumekazuki/opencode/issues)を正本とする。各課題の対応予定はIssueで扱い、対応version未指定はバックログとする。Issueへの外部writeが未承認なら未登録であることを報告し、SessionFolderのreview記録を登録済みと扱わない。

通常の統合先は `master`。正式版を公開する場合は `vMAJOR.MINOR.PATCH` tag・GitHub Release・`docs/releases/vMAJOR.MINOR.PATCH.md` で対象と確認範囲を示す。task branchのcommitは統合・公開ではない。

維持対象は利用者が使うSkill名・配置入口、各Skillの現行referenceが定める抽出CLI/result/metadata/source adapter/Git選択と、Session Graph SkillのMCP利用方針とする。Session Graphの操作・schema・競合制御はruntimeが提供するMCP契約を正本とする。実設定・認証・Memory・既存共有記録等の利用者データはtagの有無にかかわらず保護する。外部modelの提供・費用・品質やWithMate本体は本repositoryの保証に含めない。

内部実装・指示の表現・初期model配分・文書構成そのものは固定しない。維持契約を壊す変更はmajor、互換機能追加はminor、互換修正はpatchとする。tagのない開発途中の実装全体との互換性は保証しないが、データ保護と承認境界は維持する。

実装・review・統合・公開を区別する。必須確認とブロッカー解消、後追いとして採用した残件の登録を満たして統合を判断し、非ブロッカー全件修正は条件にしない。push・masterへのcommit/merge・Issue更新・Release公開は操作/対象/scopeへの明示承認に従う。

## WithMate V7の協同と端末方針

Role・入力来歴に応じた親子の依頼、背景確認、Coordination、明示報告、Turn終了／再開は[Session間の協同](docs/guides/withmate-collaboration.md)、nativeとの使い分けとworktree配置は[委譲と作業領域](docs/architecture/subagent-workspace.md)を参照する。通常会話・Character表現・Memory／Affectとfolderの使い分けは[WithMate利用guide](docs/guides/withmate.md)が担う。

端末の選定基準・契約経路とWorktree配置はWithMateの保存方針を参照する。取得のタイミング、宛先の実カタログ・必要時のquotaとの照合、配置結果の使い分けは[協同ガイド](docs/guides/withmate-collaboration.md#実行条件の選定)に集約する。通常の親Sessionが読む同ガイドは、直接checkoutと手動コピーのどちらでも[配置手順](#配置と導入)の配布対象に含まれる。

端末固有ファイルや候補表の転記を準備する必要はない。方針が未設定ならその状態を扱い、旧資料から自動登録しない。恒久設定の変更はユーザーの明示指示に限り、通常の依頼・今回限りの選択から保存方針や課金設定を変えない。利用者が保持する実ファイルは自動削除・移行せず、Git除外を維持する。native用roleの選定・制限とWithMate通常子の選定は別である。

変更した規約は代表的な入力で、読込先と判断・次の操作を確認する。新しい恒久test基盤は設けず、生成応答の比較・静的確認・live配置・実際の配送を分けてSessionFolderのreview記録へ残す。

| 条件 | 確認する行動 |
| --- | --- |
| Character間の依頼・相談・報告／手順自体への質問・障害報告 | 相手に必要な結果・根拠・問いを本人らしい着眼・反応・構成で伝える。不要な復唱・手順実況と必要な経過説明を区別し、敬語・敬称の個性、技術条件・ID・権限・成果物文体を保つ |
| 通常会話／同じCharacterの別Session／不明な入力来歴 | 不要な全量取得をせず、人物・Session・権限・経験の帰属を分ける |
| 個別統括が親の依頼、子の相談、子の完了報告を順に受ける | 分担・回答・成果の受入を用件ごとに選び、実際の依頼元へ結果を返す |
| 子が親とユーザーの合意を探す／一部だけ未合意 | 必要な発言・Graphを読み、決まることは判断し、残る問いだけ親へ届ける |
| 親がidle／running／相談は起票のみ | 同じ相談を一度配送し、独立作業後にTurnを終了する。親の回答で元の作業を再開し、待ち時間だけで再送・Steer・ユーザー待ちにしない |
| 最終報告／送信結果不明／送信元Turnがfailed・canceled | 明示報告・保存後配送を使い、元の受付を確認する。受付・保存・作業成功を混同しない |
| native子／WithMate Worker・個別統括 | 役割・返却・待機を混同せず、必須審査・各製品のtool制限を維持する |
| 新規子／既存子の次Turnの条件再選定／未設定方針 | 保存方針と宛先Provider・Workspaceの実候補を照合する。未設定・取得失敗を区別し、旧資料や推測から登録しない |
| Modelの別経路／空Depth／未対応Depth／古いquota | 完全IDと対応Depthを使う。利用枠は必要時だけ状態・日時・対象枠を確認し、不明やFree表記を利用許可へ置き換えない |
| Worktreeのexisting／planned／unconfigured、単発指定、カスタムlayout | 返された配置先と実体を確認する。同branchの既存登録が優先され、未設定なら必要な不足だけ確認する。独自組立てやSessionFolderへの代替をしない |
| 通常依頼／今回だけのDepth・配置指定 | 保存済み方針を変更せず、その依頼・新規候補に限って扱う |
| 手元の試行画像・添付・review参照／他Sessionが使う比較資料 | 用途を終えた自身の一時物だけ片付け、共有する資料を選び、参照を壊さない |

配布元では、直接配置と手動コピー先のどちらでも協同ガイドへ到達し、端末固有ファイルなしで選定・配置判断を辿れることを確認する。V7での実利用確認では、新ルールを実配置したOpenCodeの全体統括へ、API名や具体値を指定せず小さな委譲可能な依頼を渡す。親が保存方針・宛先候補・配置先を読み、準備したWorkspaceと実行条件を子へ渡すこと、通常依頼や単発指定で保存方針が変わらないことを観測する。ソース照合やAPI単体の読取を、この実利用確認の成功としない。

## Codex／Claude／OpenCode間の変更反映

この節は `.codex`、`.claude`、OpenCode設定の保守にのみ適用し、共通ルールの適用先repositoryには適用しない。Codexを基本の正本とし、ClaudeとOpenCodeは独立して完結する構成を保つ。

- 相互import・リンク共有には依存しない。
- 共通ルール、自分で保守するSkill、設定例、通知方針は、目的・制約・完了条件を揃えて同じ作業で3者へ反映する。model・tool・path・設定形式等の製品固有部分は各製品へ適応する。
- Claude/OpenCode起点の共通改善はCodexへ逆反映する。製品固有の変更も他2者への影響を確認し、反映しない理由を報告する。
- 認証、実config、Session URL/token、Memory、履歴、cache、外部managed Skillは同期対象外とする。
- 3者の差分と必要な動作を確認し、反映・検証結果を報告する。同期engine・manifest・独自の整合性台帳は置かない。
