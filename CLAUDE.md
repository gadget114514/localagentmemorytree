# localagentmemorytree — Memory Management for Claude Code

This project uses `.agent-memory/` for file-based persistent memory across sessions.

## Auto-load at Session Start

1. Read `.agent-memory/current_head.txt` to get current node path.
2. Read all `topic_summary.md` files along the path from root to current node.
3. Read the last 50 lines of the current node's `chat_history.jsonl`.

## Memory Root

Use `.agent-memory/` directory under project root. Folder naming: `NNN_PascalCase_Label` (e.g., `030_Physics_Collision`).

## Commands

### am-save — Save Memory
When user says "remember this", "save this", "record the session":
1. Read `.agent-memory/current_head.txt` for current node path. If missing, initialize with `000_Root`.
2. Append conversation turn to `.agent-memory/<node>/chat_history.jsonl`.
3. Update decisions/open issues in `.agent-memory/<node>/topic_summary.md`.
4. If topic changed, create child folder and update `current_head.txt`.
5. Add entry to `.agent-memory/index.md` for new nodes.

### am-recall — Recall Memory
When user says "remind me", "what was last time?", "tell me about X":
1. Read `.agent-memory/index.md` for node list.
2. Read `topic_summary.md` of keyword-related nodes.
3. If needed, read last 50 lines of `chat_history.jsonl`.
4. Summarize findings.

### am-set-topic — Set Topic Label
When user says "set topic name", "label this", "change the title":
1. Read `current_head.txt`.
2. Use user-specified label or infer from context.
3. Save as `title:` in `topic_summary.md` frontmatter.

### am-forget — Delete Memory
When user says "forget this", "delete", "no longer needed":
1. Confirm target node with user.
2. Move folder to `.agent-memory/_archived/`.
3. Remove entry from `index.md`.
4. If `current_head.txt` pointed to deleted node, return to parent.

### am-stack — Display Stack
When user says "show stack", "stack contents", "pending":
1. Read `.agent-memory/_stack.txt` and display entries.

### am-pop / am-resume — Restore from Stack
When user says "resume", "go back", "restore from stack":
1. Pop last line from `_stack.txt`.
2. Update `current_head.txt` to that path.
3. Read and display `topic_summary.md`.

### am-push / am-suspend — Save to Stack and Start New
When user says "suspend", "stack this", "come back later":
1. Read `current_head.txt` for current path.
2. Append to `_stack.txt`.
3. Confirm new topic name with user.
4. Create child folder (`NNN_PascalCase_Label`), `topic_summary.md`, `chat_history.jsonl`.
5. Update `current_head.txt` and `index.md`.

### am-change-topic — Move to Topic
When user says "move to XXX", "switch to XXX":
1. Confirm destination with user.
2. Verify from `index.md`, update `current_head.txt`, display `topic_summary.md`.

### am-list — Child Topics
When user says "child topics", "show children":
1. Enumerate child folders under current node.
2. Read and display purpose/status from each `topic_summary.md`.

### am-up — Move to Parent
When user says "back to parent", "move up":
1. Move one level up from current path. Update `current_head.txt`.
2. Display parent `topic_summary.md`.

### am-remove-topic — Delete Specified Topic
When user says "remove topic", "delete XXX":
1. Confirm with user, verify from `index.md`.
2. Move folder to `_archived/`, remove from `index.md`.

### am-move-topic — Move Under Another Topic
When user says "move topic", "move XXX under YYY":
1. Confirm both nodes, move source folder under destination.
2. Update `index.md` and `current_head.txt` if needed.

### am-make-topic — Create Child Topic
When user says "new topic", "create child topic":
1. Confirm name, create child folder with `topic_summary.md` and `chat_history.jsonl`.
2. Update `current_head.txt` and `index.md`.

### am-current / am-where — Display Current Topic
When user says "current topic", "where are we":
1. Read and display `topic_summary.md` of current node.

### am-list-all-topic — Memory List
When user says "list all", "show all", "what I remember":
1. Read `index.md` and display full tree with statuses.
