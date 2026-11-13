# WithMate Memory／Character 接続

接続設定・障害調査時に読む。利用判断・記録範囲・継続的な許可は[WithMate利用guide](../guides/withmate.md)、操作名・引数・保存先・再試行は現在のMCP instructions/tool schemaを正本とする。

## managed HTTP

WithMateのOpenCode対応providerは、native global/project configを保持した専用CLI serverへSDKで `withmate-memory` と `withmate-glossary` をturnごとに追加する。Memory endpointは `/mcp/memory`。接続設定の `type: "remote"`、`url`、`headers`、`oauth: false` はhostが供給する。

`x-withmate-mcp-token` はMain管理の短命token。URL・token・runtime bindingを `opencode.jsonc`、template、logへ固定保存しない。Desktopの既存serviceを停止したり、そのpasswordを流用したりしない。WithMateのOpenCode対応版と、実際のsession bindingが必要である。

Character Definition・Output Boundary・Workspace・SessionFolder・Additional Directories・Affect contextは共通promptで渡される。Character表現の適用境界は[WithMate利用guide](../guides/withmate.md#character表現の適用範囲)に従い、応答と非公開の手元の自然文へ適用する。repository内・外部向け成果物は明示指定がない限り既存文体と目的を優先する。共通AGENTSへ特定Characterのsnapshotを固定しない。

## 確認と障害

1. WithMateからOpenCode sessionを開始し、現在提供されたMCP toolsと必要なread-only操作で接続を確認する。CLIの永続登録一覧だけでは一時接続を判定しない。
2. provider/CLI version、起動元、sessionとbindingの対応、設定読込元を確認する。認証値の全文は出力しない。
3. binding不足・token失効はhost側の供給を確認する。他sessionの値や推測値で補わず、通常CLIからWithMate内部DBへ直接アクセスしない。

接続確認のためにMemory/Affectを作らない。通常会話での能動的なMemory読取、背景実行による自動生成、明示依頼による保存は[WithMate利用guide](../guides/withmate.md)に従う。接続成功だけで保存済みとせず、失敗時も独自storeへ逃がさない。

## 背景登録と添付の診断

背景処理の実行結果と操作診断はWithMateの`userData/logs/background-failures.jsonl`で、Agentの終了状態と個々のtool操作の成否・保存効果を分けて読む。`completed`は全件保存成功の証明ではなく、操作拒否の併存だけで実行全体を`failed`へ読み替えない。全操作拒否も正常な登録不要とは扱わない。

このログには正常終了した実行の操作拒否・失敗・結果不明も記録される。ログの存在だけから実行失敗を推定せず、先行する保存を巻き戻したり、実行全体を無条件に再送したりしない。

MCP／CLIが`WITHMATE_MEMORY_INVALID_INPUT`を返した場合は、`error.details.tool`と`error.details.issues`のfield位置・制約・代替条件を読んで入力を修正し、呼び直す。unionの候補内の条件はすべて必要だが、候補間は代替関係であり、全候補を同時に満たす必要はない。公開schemaでの拒否は当該呼出しが保存前に拒否されたことを示し、`effect: none`を他の操作の保存まで未実施という意味にしない。

この入力拒否を認可・通信・保存の障害やCLI fallbackの許可へ読み替えず、変更なしの要求を繰り返さない。Memory本文・会話本文・prompt・tool引数や応答本文・入力値全体・secret・認証情報を追加dumpしない。

背景thread・前回失敗・待機状態はアプリ再起動をまたいで復元されないが、保存済みMemory・Affectは保持される。threadの継続や失敗情報の有無から保存状態を推測しない。

添付の取込元・復元先とローカルwriteの可否は当該呼出しの許可範囲で確認する。復元の拒否をMCP全体の未接続と同一視したり、権限を緩めて迂回したりしない。適用境界は[WithMate利用guide](../guides/withmate.md)に従う。

## 手動STDIO診断

WithMateが配置した `withmate-memory mcp-server` を用いる診断経路は、当該Sessionのbindingを正しく継承できる場合だけ使う。V2では `mcp.servers.<name>` の `type: "local"` と `command` 配列を使い、local processは親の環境を継承する。Codexの `env_vars` は転記しない。Windowsのshim起動は `["cmd.exe", "/d", "/c", "withmate-memory", "mcp-server"]`。

必要なbinding環境は `WITHMATE_AGENT_RUNTIME_BINDING_REFERENCE`、`WITHMATE_AGENT_RUNTIME_BINDING_REQUIRED`、`WITHMATE_AGENT_RUNTIME_TURN_CAPABILITY`、`WITHMATE_MEMORY_RUNTIME_APPLICATION_INSTANCE_ID`、`WITHMATE_MEMORY_RUNTIME_GENERATION_ID`。実値はhost管理とし、欠落を手入力で偽装しない。この診断経路を標準設定へ常設したりmanaged HTTPと重複登録したりしない。HTTP失敗時の自動fallbackではない。

[OpenCode V2 MCP仕様](https://opencode.ai/v2/docs/mcp-servers/)
