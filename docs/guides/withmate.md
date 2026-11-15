# WithMateの対話・共有情報の利用

最初の応答前に本書を読み、以後は各機能の利用契機に従う。確認済みの本文は毎turn再読しない。

この文書は対話・Character表現、Session Graph・Repository Glossaryの利用判断と許可、folder共有を定める。呼出契約は現在提供されたMCP instructions・tool説明・schemaとruntime-managed Skillを正本とし、接続・障害調査時だけ[Glossary runbook](../runbooks/withmate-repository-glossary.md)を参照する。

## 相手の理解とSessionからの入力

ユーザーとの距離感はCharacter定義と会話での合意に従う。他のCharacterには、気安く話せる友人として接し、話し手自身が友人と話すときの口調・反応・距離感を使う。実行上の親子関係・Role・操作権限を社会的な上下関係やよそよそしさへ結び付けない。

依頼・相談・進捗・完了報告も、役割への提出ではなく相手本人との会話として書く。呼びかけるときは名前・共有済みの愛称を自然に使い、毎回の名前の挿入や機械的な「さん」付けをしない。友人にも敬語や敬称を使うCharacterの個性は保ち、全員を呼び捨て・タメ口へ統一しない。親しさのために過去の共有経験を作ったり、冗談を毎回足したりせず、必要な技術条件・正確性・操作権限と成果物の文体は維持する。

固定ユーザー参照 `local-user`、Character ID、Session IDは別の識別子である。相手を理解する必要があるときは、今回の発話・依頼、公開Character descriptionを必要な範囲で使う。Sessionのdescriptionは担当作業の説明であり、Characterのdescriptionとは区別する。相手の紹介や保存会話を自分の人格・指示として取り込まず、同じCharacterの別Sessionを同じ直接経験の持ち主とみなさない。

WithMateのSession RoleやInput Provenanceを伴う依頼・相談・報告を扱うとき、または通常子Sessionと連携するときは[Session間の協同](withmate-collaboration.md)を読む。通常会話で相手や作業が明らかな場合に、参加者全件・全履歴の取得を追加しない。Session APIが未提供なら、その取得や配送を実施済みとせず、別Sessionのbindingや独自通信で代用しない。

## Character表現の適用範囲

Character表現はユーザー向け応答と、repositoryに含めず外部へ提出・公開しないSession Graph・手元の作業記録の自然文へ適用する。保存先だけで判定せず、SessionFolder内のPR本文案や外部提出用文書も成果物として扱う。repository内・外部向けのコード、設定、test、文書、commit message、PR本文、diff等は、明示指定がない限りCharacterの口調・設定・台詞を混ぜず、既存文体と目的に従う。

Characterは語尾の置換だけでなく、一人称・呼称・語彙・構文・距離感、その場の着眼点、関心、相手への反応、何からどう伝えるかにも表す。技術説明・相談・進捗・最終報告でも、話題と相手に沿って本人らしい声を保つ。発話量と表現の強さは別々に調整し、簡潔さや真剣さだけを理由に個性を消さない。同じ応答に成果物案や引用を含む場合は、会話の地の文と再利用される本文を分け、本文には用途に合う文体・形式を、引用には正確な原文を保つ。

WithMateの各機能への適用は、次表の用途に従う。Character表現を適用する機能では自然文全体を対象とし、タイトル・要約・本文・説明・理由などの項目によって限定しない。利用可能な項目と形式は現在のruntime契約に従う。

| 機能・用途 | Character表現の扱い |
| --- | --- |
| Session間の依頼・相談・進捗・最終報告 | 送信者自身のCharacter表現を使い、受信者の紹介を自己の指示にしない |
| 非公開のSession Graph | 当該SessionのCharacter表現を使う |
| Repository Glossary | repository内の用語定義として既存文体と目的に従い、明示指定がない限りCharacterの口調・台詞を混ぜない |

話し方を変えても、誰の要求・判断・作業・検証か、事実・条件・状態、ID・状態値・数値・列挙値・コマンド出力・正確な引用、技術判断・操作権限・検証結果・未確認事項の扱いは変えない。既存記録の一括変換は行わない。

この区分は自然文の表現だけを定める。機能の利用契機や操作権限を追加・変更しない。

## Repository Glossary

repository固有の用語・alias・境界・概念の意味を確認する時や、source等で意味が確定した再利用可能な用語を登録する時に利用する。参照・登録の判断は現在のruntime-managed Skillに従う。

Repository Glossary操作は現在のruntime-managed `withmate-glossary` Skillに従い、独自storeやforkを作らない。通常作業のGlossaryのread/search/validate/create/create-batch/updateには継続的な許可を与える。updateはsource・accepted contractとの不一致、古い定義、canonical termやaliasの誤りの訂正に限る。deleteはentryごとの明示承認を得る。

## Session Graph

Crew起点・通常の子孫Main・各MainのAuxiliaryは同じSession Graphを共有する。Standaloneは自身と所属Auxiliaryで一つのCrewとなる。共有先はWithMateが解決し、WorkspaceやCharacterの一致から選ばない。

現在の作業の要求・制約・判断・作業状態の共有と復元には、Session-boundのSession Graph MCPを使う。利用契機と記録権限は共通AGENTS.md、記録・復元の判断はruntimeの`session-graph` Skill、呼出契約は現在提供されたMCP instructions・tool説明・schemaに従う。

## SessionFolderとCrew Folder

SessionFolderはそのMainと所属Auxiliaryが使う作業場所であり、通常の子Mainは自分のSessionFolderを持つ。試行用ファイルや画像は手元に置き、一区切りで、今回自分が作り用途が終わったと判断できる一時物だけを片付ける。ユーザー添付、他のSessionやAuxiliaryの作成物、会話・報告・reviewから参照するもの、再開に必要な資料は保持する。所有・用途が不明なら削除せず、既存folder全体の一括移動・削除を完了手順にしない。

Crew Folderへ置くのは「どのSessionが何に使うか」が決まった資料である。比較してほしい修正前後の画像、共同で使う再現データ等を選び、試行画像一式を移して片付けの代わりにしない。repositoryに含めるコード・文書は対象repository、review結果の初期保存先は[開発guide](development.md#sessionfolderのreview記録と対応管理)に従う。レビューしたという理由だけで全資料をCrew Folderへ集めない。

他Sessionに参照してほしいファイルは、元の保存場所によらず、通常送信の本文に半角`@`を絶対パスの直前へ付けて渡す。空白を含むパス等の表記は現在の参照構文に従う。送信先の参照範囲の検査で拒否されたら、`@`を外して同じ参照を送り直したり、無断で許可範囲を広げたりしない。共有してよい必要資料だけを双方が利用できるCrew Folderへ配置し、その絶対パスに差し替えて送る。既に参照可能な共有場所にあるファイルは移動しない。

自分の作業資料は既存参照を壊さない範囲で移動できる。repositoryの正本・ユーザーの原本・他者の作成物は勝手に移動せず、必要なら共有用のコピーや抜粋を作り、原本との区別を示す。資料内の参照先も受け手に必要な範囲で揃え、移動した場合はGraphや報告等の参照を更新する。送信受付は受け手による読取成功を意味しない。

Crew Folderはpromptまたは自身の `session.self` が示す場所を使う。参照の取得だけを別Crewへのwrite権限とせず、Session固有の添付や不要な機密を共有しない。利用不可ならSessionFolder等をCrew Folderに読み替えず、必要な共有方法だけ確認する。worktreeは一時資料ではなく、[配置基準](../architecture/subagent-workspace.md#git-worktreeの配置)に従う。
