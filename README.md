# localagentmemorytree

Local file-based memory management for AI coding agents.
No database or external tools required — works with **pure file operations**.

## Features

- **File-based** — stores everything in `.agent-memory/` directory. Git-friendly.
- **Agent-agnostic** — share the same memory across opencode and Claude Code interchangeably.
- **Tree structure** — folder hierarchy doubles as a mind map.
- **Stack mechanism** — suspend/resume topics for seamless context switching.

## Quick Install

### opencode

1. **Per-project setup (recommended)** — append the contents of `AGENTS.md` to your project's `AGENTS.md`. This project already includes the rules at the repository root.
2. **Slash commands** — copy the command files to enable `/am-save`, `/am-recall`, etc.:
   ```bash
   cp skills/commands/* ~/.config/opencode/commands/
   ```

### Claude Code

Copy the command files as Claude Code skills:

```bash
cp skills/commands/* ~/.claude/plugins/localagentmemorytree/
```

Claude Code will match natural-language triggers (e.g. "remember this", "what did we do last time") and route them to the correct operation automatically.

**For detailed Claude Code setup and complete usage guide, see [README_CLAUDE_CODE.md](README_CLAUDE_CODE.md).**

## Usage

Commands can be triggered via natural language or `/am-` slash commands.

| Command | Alias | Triggers |
|---|---|---|
| `am-save` | | "remember this", "save this session", "keep in memory" |
| `am-recall` | | "what did we do last time", "remind me about" |
| `am-forget` | | "forget this", "delete this topic" |
| `am-set-topic` | | "set topic name", "label this topic" |
| `am-make-topic` | | "new topic", "add subtopic" |
| `am-remove-topic` | | "remove topic", "delete XXX" |
| `am-change-topic` | | "switch to XXX", "go to XXX" |
| `am-move-topic` | | "move topic", "move XXX under YYY" |
| `am-current` | `am-where` | "current topic", "where am I" |
| `am-up` | | "go to parent", "move up" |
| `am-list-all-topic` | | "list all", "show memory" |
| `am-list` | | "show subtopics", "children" |
| `am-push` | `am-suspend` | "suspend", "push to stack" |
| `am-pop` | `am-resume` | "resume", "pop from stack" |
| `am-stack` | | "show stack", "stack contents" |

## Storage Structure

```
<project_root>/.agent-memory/
├── current_head.txt          # Current node path
├── index.md                  # Full node index
├── _stack.txt                # Suspension stack
└── 000_Root/
    ├── topic_summary.md       # Project goal & decisions
    ├── 010_Feature_X/
    │   ├── topic_summary.md
    │   └── chat_history.jsonl # Conversation history (JSONL)
    └── 020_Feature_Y/
        ├── topic_summary.md
        └── chat_history.jsonl
```

Folder naming convention: `NNN_PascalCase_Label` (e.g. `030_Physics_Collision`)

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

## Design Documents

- `AGENTS.md` — Project-level memory management rules
- `DESIGN.md` — Full specification (all command details)

## License

MIT
