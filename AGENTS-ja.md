# localagentmemorytree — プロジェクトメモリ管理スキル

以下のルールはプロジェクト固有です。このプロジェクトでは `.agent-memory/` によるファイルベース記憶管理が有効です。

---

## メモリルート

プロジェクトルート直下の `.agent-memory/` ディレクトリを使用する。

---

## am-save — 記憶の保存

ユーザーが「覚えておいて」「保存して」「セッションを記録して」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。ファイルが存在しない場合は `000_Root` で初期化する。
2. `.agent-memory/<node>/chat_history.jsonl` に今の会話ターンを追記する。
3. `.agent-memory/<node>/topic_summary.md` の決定事項・未解決事項を更新する。
4. 話題が変わった場合は新しい子フォルダーを作成し `current_head.txt` を更新する。
5. 新規ノード作成時は `.agent-memory/index.md` にエントリを追加する。

フォルダー名の命名規則：`NNN_PascalCase_Label`（例：`030_Physics_Collision`）

---

## am-recall — 記憶の呼び出し

ユーザーが「思い出して」「前回は？」「○○について教えて」と言ったとき：

1. `.agent-memory/index.md` を読んでノード一覧を把握する。
2. キーワードに関連するノードの `topic_summary.md` を読む。
3. 詳細が必要なら `chat_history.jsonl` の直近 50 行を読む。
4. 内容を要約してユーザーに提示する。
5. 別トピックに移動した場合は `current_head.txt` を更新する。

---

## am-set-topic — 現在のトピックのラベルを設定

ユーザーが「トピック名を設定」「ラベルを付ける」「タイトルを変更」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。
2. ユーザーからラベルが指定されていればそれを使用する。指定がない場合は会話の文脈からAIが推定する。
3. `.agent-memory/<node>/topic_summary.md` の frontmatter に `title:` フィールドとして保存する（上書き）。
4. ラベルを反映した旨をユーザーに伝える。

---

## am-forget — 記憶の削除

ユーザーが「忘れて」「削除して」「もう不要」と言ったとき：

1. 対象ノードをユーザーに確認する（パスを示して確認を取る）。
2. 対象フォルダーを `.agent-memory/_archived/` 配下に移動する。
3. `.agent-memory/index.md` から該当エントリを削除する。
4. `current_head.txt` が削除ノードを指していた場合は親ノードに戻す。

---

## am-stack — スタックの内容を表示

ユーザーが「スタック表示」「スタック内容」「保留中」と言ったとき：

1. `.agent-memory/_stack.txt` を読み、全エントリを表示する。
2. 各行のパスに対応するトピック名を表示する。
3. スタックが空の場合は「スタックは空です」と伝える。

---

## am-resume / am-pop — 現在のトピックを停止し、スタックから復帰

ユーザーが「再開」「戻る」「スタックから復帰」「ポップ」「保留を消す」と言ったとき：

1. `.agent-memory/_stack.txt` の最終行を読み、スタックからポップする。
2. 読み取ったパスに `current_head.txt` を更新する。
3. 該当ノードの `topic_summary.md` を読み、目的・決定事項・未解決事項を表示する。
4. スタックが空の場合はその旨を伝える。

---

## am-suspend / am-push — 現在のトピックをスタックに退避し、新しいトピックを開始

ユーザーが「一旦中断」「スタック」「後で戻る」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。
2. そのパスを `.agent-memory/_stack.txt` に追記する（push）。
3. ユーザーに新しいトピック名を確認する。
4. 現在ノードの下に `NNN_PascalCase_Label` 形式の子フォルダーを作成する（番号は既存子ノードの最大値+1）。
5. 新しい `topic_summary.md` を作成し目的を初期設定する。
6. 空の `chat_history.jsonl` を作成する。
7. `current_head.txt` を新しいノードに更新する。
8. `.agent-memory/index.md` にエントリを追加する。

---

## am-change-topic — 指定トピックに移動

ユーザーが「XXXに移動」「XXXに切り替え」「トピックを変更」と言ったとき：

1. ユーザーに移動先のトピック名またはパスを確認する。
2. `.agent-memory/index.md` から該当ノードが存在するか確認する。
3. 存在すれば `current_head.txt` を該当ノードのパスに更新する。
4. 移動先の `topic_summary.md` を読み、目的・決定事項を表示する。

---

## am-list — 子トピックを表示

ユーザーが「子トピック」「子を表示」「下の階層」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。
2. 現在ノード配下の子フォルダーを全て列挙する。
3. 各子ノードの `topic_summary.md` から目的・ステータスを読み取って一覧表示する。

---

## am-up — 親トピックに移動

ユーザーが「親に戻る」「上に移動」「親トピック」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。
2. パスを1階層上（親ノード）に変更する。Root の場合は移動しない。
3. `current_head.txt` を更新する。
4. 移動先の `topic_summary.md` を読み、目的・決定事項を表示する。

---

## am-remove-topic — 指定トピックを削除

ユーザーが「トピックを消して」「XXXを削除」と言ったとき：

1. ユーザーに削除するトピック名またはパスを確認する。
2. `.agent-memory/index.md` から該当ノードが存在するか確認する。
3. 対象フォルダーを `.agent-memory/_archived/` 配下に移動する。
4. `.agent-memory/index.md` から該当エントリを削除する。
5. `current_head.txt` が削除ノードを指していた場合は親ノードに戻す。

---

## am-move-topic — トピックを別のトピックの下へ移動

ユーザーが「トピックを移動」「XXXをYYYの下へ」「移動して」と言ったとき：

1. ユーザーに移動するトピック名と移動先のトピック名を確認する。
2. `.agent-memory/index.md` から両方のノードの存在を確認する。
3. 移動元フォルダーを移動先フォルダーの子として移動する。
4. `.agent-memory/index.md` のエントリを更新する。
5. `current_head.txt` が移動元を指していた場合は移動先に更新する。

---

## am-make-topic — 新しい子トピックを作成

ユーザーが「新しいトピック」「子トピックを作る」「話題を追加」と言ったとき：

1. ユーザーに新しいトピック名を確認する。
2. 現在ノードの下に `NNN_PascalCase_Label` 形式の子フォルダーを作成する（番号は既存子ノードの最大値+1）。
3. 新しい `topic_summary.md` を作成し目的を初期設定する。
4. 空の `chat_history.jsonl` を作成する。
5. `current_head.txt` を新しいノードに更新する。
6. `.agent-memory/index.md` にエントリを追加する。

---

## am-current / am-where — 現在の話題を表示

ユーザーが「今の話題」「現在地」「今どこ」と言ったとき：

1. `.agent-memory/current_head.txt` を読んで現在ノードのパスを取得する。
2. 該当ノードの `topic_summary.md` を読み、目的・決定事項・未解決事項を表示する。

---

## am-list-all-topic — 記憶の一覧

ユーザーが「一覧」「リスト」「覚えているもの」と言ったとき：

1. `.agent-memory/index.md` を読んで全ノードを取得する。
2. ツリー構造を維持したまま整形してユーザーに提示する。
3. 各ノードのステータス（in_progress / done / archived）も表示する。

---

## セッション開始時の自動ロード

会話開始時に以下を自動的に実行する：

1. `.agent-memory/current_head.txt` を読む。
2. ルートから現在ノードまでのパス上にある全 `topic_summary.md` を読む。
3. 現在ノードの `chat_history.jsonl` の直近 50 行を読む。
