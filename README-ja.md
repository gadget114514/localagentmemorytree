# localagentmemorytree

エージェントのメモリをローカルのファイルとして保存・管理するスキル。
DB・外部ツール不要で**ファイル操作だけ**で完結する。

## Features

- **ファイルベース** — `.agent-memory/` ディレクトリに保存。Git管理可能
- **エージェント非依存** — opencode / Claude Code を交互に使っても同じ記憶を共有
- **ツリー構造** — フォルダー階層がそのままマインドマップ
- **スタック機構** — トピックの中断・復帰でコンテキストスイッチ

## Quick Install

### opencode

```bash
# グローバルルール（全プロジェクトで有効）
cp -r rules/opencode/localagentmemorytree.md ~/.config/opencode/rules/

# スラッシュコマンド（全プロジェクトで有効）
cp commands/* ~/.config/opencode/commands/
```

またはプロジェクトルートの `AGENTS.md` に `AGENTS.md` の内容を追記。

### Claude Code

```bash
cp -r skills/claude/* ~/.claude/plugins/localagentmemorytree/
```

## Usage

コマンドは自然言語トリガーまたは `/am-` スラッシュコマンドで実行します。

| Command | Alias | Triggers |
|---|---|---|
| `am-save` | | 覚えておいて、保存して、記録して |
| `am-recall` | | 思い出して、前回は？、教えて |
| `am-forget` | | 忘れて、削除して、もう不要 |
| `am-set-topic` | | トピック名を設定、ラベルを付ける |
| `am-make-topic` | | 新しいトピック、話題を追加 |
| `am-remove-topic` | | トピックを消して |
| `am-change-topic` | | XXXに移動、切り替え |
| `am-current` | `am-where` | 今の話題、現在地、今どこ |
| `am-up` | | 親に戻る、上に移動 |
| `am-list-all-topic` | | 一覧、リスト |
| `am-list` | | 子トピック、子を表示 |
| `am-move-topic` | | トピックを移動、XXXをYYYの下へ |
| `am-suspend` | `am-push` | 一旦中断、スタック |
| `am-resume` | `am-pop` | 再開、戻る、ポップ |
| `am-stack` | | スタック表示、保留中 |

## Storage Structure

```
<project_root>/.agent-memory/
├── current_head.txt          # 現在のノードパス
├── index.md                  # 全ノードのインデックス
├── _stack.txt                # 中断スタック
└── 000_Root/
    ├── topic_summary.md       # プロジェクトの目的・決定事項
    ├── 010_Feature_X/
    │   ├── topic_summary.md
    │   └── chat_history.jsonl # 会話履歴（JSONL）
    └── 020_Feature_Y/
        ├── topic_summary.md
        └── chat_history.jsonl
```

フォルダー命名規則：`NNN_PascalCase_Label`（例: `030_Physics_Collision`）

## Architecture

```
┌─────────────────────────────────────────────────────┐
│               Integration Layer (Agent-specific)     │
│  opencode: ~/.config/opencode/rules/                │
│  Claude Code: ~/.claude/plugins/                    │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│              Storage Layer (Agent-agnostic)           │
│              <project_root>/.agent-memory/            │
└─────────────────────────────────────────────────────────┘
```

## Design Docs

- `second.md` — 設計書（アーキテクチャ、ファイル仕様）
- `third.md` — 最終仕様書（全コマンド詳細）

## License

MIT
