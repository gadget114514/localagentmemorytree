# localagentmemorytree — Final Specification

## 1. Overview

A skill for saving and managing agent memory as local files.
Complete with only file operations, no DB or external tools required.

### Target Agents

| Agent | Integration Method |
|---|---|
| **opencode** | Injected as prompt rules in `~/.config/opencode/rules/localagentmemorytree.md` |
| **Claude Code** | Registered as a skill in `~/.claude/plugins/localagentmemorytree/skills/*/SKILL.md` |

### Implementation Files

| File | Path |
|---|---|
| Skill Definition | `~/.agents/skills/localagentmemorytree/SKILL.md` |
| opencode Rules | `~/.config/opencode/rules/localagentmemorytree.md` |

Design principle: File formats (directory, .md, .jsonl) are agent-agnostic and standardized.
The same project memory can be shared even when alternating between opencode and Claude Code.

---

## 2. Storage Structure

### Memory Root

`.agent-memory/` directly under the project root

```
.agent-memory/
├── current_head.txt          # Current node path
├── index.md                  # All nodes index
├── _stack.txt                # Suspend stack (1 entry per line)
└── 000_Root/
    ├── topic_summary.md
    ├── 010_Feature_X/
    │   ├── topic_summary.md
    │   └── chat_history.jsonl
    └── 020_Feature_Y/
        ├── topic_summary.md
        └── chat_history.jsonl
```

### Folder Naming Convention

`NNN_PascalCase_Label` (e.g., `030_Physics_Collision`)

### Node Hierarchy

| Layer | Role | Entity |
|---|---|---|
| Root | Project-wide objectives and basic policies | `000_Root/topic_summary.md` |
| Branch | Specific feature or problem area | `010_Feature_X/topic_summary.md` |
| Leaf | Specific conversations and decisions within a branch | `010_Feature_X/chat_history.jsonl` |

---

## 3. File Specification

### topic_summary.md

```markdown
---
node_id: 010_Unity_ECS
parent: 000_Root
status: in_progress   # in_progress | done | archived
updated: 2026-05-14T10:30:00
---

## Purpose
Design and implementation of a terrain generation system using Unity ECS

## Decisions
- Chunk size fixed to 16x16
- Use IJobEntity for jobification

## Open Issues
- LOD switching timing
```

### chat_history.jsonl

```jsonl
{"turn": 1, "timestamp": "2026-05-14T10:00:00", "role": "user", "content": "I want to generate terrain with ECS"}
{"turn": 2, "timestamp": "2026-05-14T10:00:05", "role": "assistant", "content": "Using IJobEntity is the optimal approach."}
```

### current_head.txt

A single-line file holding the current node path.

```
000_Root/010_Unity_ECS/011_System_Job
```

### index.md

```markdown
# Memory Index

- [000_Root](000_Root/topic_summary.md) — Project basic policy `done`
  - [010_Unity_ECS](000_Root/010_Unity_ECS/topic_summary.md) — ECS terrain generation `in_progress`
    - [011_System_Job](000_Root/010_Unity_ECS/011_System_Job/topic_summary.md) — Jobification `in_progress`
  - [020_Asset_Workflow](000_Root/020_Asset_Workflow/topic_summary.md) — Asset management `done`
```

### _stack.txt

Suspend stack. Appended 1 entry per line. Push with am-suspend, pop with am-resume/am-pop.

```
000_Root/010_Unity_ECS
000_Root/020_Asset_Workflow
```

---

## 4. Command List

| Command | Alias | Trigger Phrases |
|---|---|---|
| **am-set-topic** | — | Set topic name, add label, change title |
| **am-save** | — | Remember this, save, record session |
| **am-recall** | — | Remind me, what was last time?, tell me about X |
| **am-forget** | — | Forget, delete, no longer needed |
| **am-make-topic** | — | New topic, create child topic, add subject |
| **am-remove-topic** | — | Delete topic, remove XXX |
| **am-change-topic** | — | Move to XXX, switch to XXX, change topic |
| **am-current** | am-where | Current topic, current location, where are we |
| **am-up** | — | Back to parent, move up, parent topic |
| **am-list-all-topic** | — | List, all topics, what I remember |
| **am-list** | — | Child topics, show children, lower hierarchy |
| **am-move-topic** | — | Move topic, put XXX under YYY, relocate |
| **am-suspend** | am-push | Suspend, stack, come back later |
| **am-resume** | am-pop | Resume, go back, pop, clear pending |
| **am-stack** | — | Show stack, stack contents, pending |

---

## 5. Command Details

### am-set-topic — Set the current topic label

1. Read `current_head.txt` to get the current node path.
2. Use the label if specified by the user; otherwise, AI infers from context.
3. Overwrite save as `title:` field in the frontmatter of `<node>/topic_summary.md`.
4. Inform the user that the label has been applied.

### am-save — Save Memory

1. Read `current_head.txt` to get the current node path. If it doesn't exist, initialize with `000_Root`.
2. Append the current conversation turn to `<node>/chat_history.jsonl`.
3. Update the decisions and open issues in `<node>/topic_summary.md`.
4. If the topic has changed, create a new child folder and update `current_head.txt`.
5. Add an entry to `index.md` when creating a new node.

### am-recall — Recall Memory

1. Read `index.md` to get the full node list.
2. Read `topic_summary.md` of nodes related to the keyword.
3. If needed, read the last 50 lines of `chat_history.jsonl`.
4. Summarize and present to the user.
5. If switching to a different topic, update `current_head.txt`.

### am-forget — Delete Memory

1. Confirm the target node with the user (show the path).
2. Move the target folder to `_archived/`.
3. Remove the corresponding entry from `index.md`.
4. If `current_head.txt` pointed to the deleted node, reset to parent node.

### am-move-topic — Move topic under another topic

1. Confirm with the user the topic to move and the destination topic name.
2. Verify both nodes exist from `index.md`.
3. Move the source folder as a child of the destination folder.
4. Update entries in `index.md`.
5. If `current_head.txt` pointed to the source, update to destination.

### am-make-topic — Create a new child topic

1. Confirm the new topic name with the user.
2. Create a child folder under the current node in `NNN_PascalCase_Label` format (number is max existing child number + 1).
3. Create `topic_summary.md` and initialize the purpose.
4. Create an empty `chat_history.jsonl`.
5. Update `current_head.txt` to the new node.
6. Add an entry to `index.md`.

### am-remove-topic — Delete a specific topic

1. Confirm the topic name or path to delete with the user.
2. Verify the node exists from `index.md`.
3. Move the target folder to `_archived/`.
4. Remove the corresponding entry from `index.md`.
5. If `current_head.txt` pointed to the deleted node, reset to parent node.

### am-change-topic — Move to a specified topic

1. Confirm the destination topic name or path with the user.
2. Verify the node exists from `index.md`.
3. If it exists, update `current_head.txt` to that path.
4. Read and display the destination `topic_summary.md`.

### am-current / am-where — Display current topic

1. Read `current_head.txt` to get the current node path.
2. Read and display `topic_summary.md` of that node.

### am-up — Move to parent topic

1. Read `current_head.txt` to get the current node path.
2. Move the path up one level. Don't move if already at Root.
3. Update `current_head.txt`.
4. Read and display the destination `topic_summary.md`.

### am-list-all-topic — Memory list

1. Read `index.md` to get all nodes.
2. Format and present while maintaining the tree structure.
3. Also display the status of each node.

### am-list — Display child topics

1. Read `current_head.txt` to get the current node path.
2. Enumerate all child folders under the current node.
3. Read the purpose and status from each child node's `topic_summary.md` and display as a list.

### am-suspend / am-push — Save the current topic to stack and start a new topic

1. Read `current_head.txt` to get the current node path.
2. Append that path to `_stack.txt` (push).
3. Confirm the new topic name with the user.
4. Create a child folder under the current node (number is max existing child number + 1).
5. Create a new `topic_summary.md` and empty `chat_history.jsonl`.
6. Update `current_head.txt` to the new node.
7. Add an entry to `index.md`.

### am-resume / am-pop — Stop current topic and restore from stack

1. Read the last line of `_stack.txt` and pop from the stack.
2. Update `current_head.txt` to the read path.
3. Read and display `topic_summary.md` of that node.
4. If empty, inform the user "Stack is empty."

### am-stack — Display stack contents

1. Read `_stack.txt` and display all entries.
2. Display the topic name corresponding to each line path.
3. If empty, inform the user "Stack is empty."

---

## 6. Auto-load on Session Start

Auto-execute the following at conversation start:

1. Read `current_head.txt`.
2. Read all `topic_summary.md` files along the path from root to current node.
3. Read the last 50 lines of the current node's `chat_history.jsonl`.

### Context Load Priority

1. `current_head.txt` (verify current location)
2. All `topic_summary.md` along root → current node path
3. Current node's `chat_history.jsonl` (last N turns only)
4. Sibling/child node `topic_summary.md` as needed

---

## 7. Branch Judgment Logic

When the topic changes during a session, the AI autonomously decides.

```
New utterance is...
  ├─ Deep dive / elaboration on current node → Stay (record in current folder)
  ├─ Different feature / issue           → Branch (create new child folder)
  └─ Completely different project area → Return to Root then Branch
```

---

## 8. Implementation Phases

| Phase | Description | Status |
|---|---|---|
| 1 | Basic save/recall operations (flat single layer) | ✅ |
| 2 | Branch judgment logic (auto Branch) | ✅ |
| 3 | forget (archive function) | ✅ |
| 4 | Automatic consistency check of index.md | ⏳ Not started |

---

## 9. Architecture

```
┌──────────────────────────────────────────────────────────┐
│              Integration Layer (Agent-specific)           │
│                                                          │
│  opencode                          Claude Code            │
│  ~/.config/opencode/rules/         ~/.claude/plugins/    │
│    localagentmemorytree.md               localagentmemorytree/   │
│                                      skills/*/SKILL.md   │
└──────────────────────┬───────────────────────────────────┘
                       │ Execute same file operations
┌──────────────────────▼───────────────────────────────────┐
│             Storage Layer (Common, Agent-agnostic)         │
│  <project_root>/.agent-memory/                           │
│    current_head.txt / index.md / _stack.txt              │
│    000_Root/topic_summary.md                             │
│    010_Feature_X/{topic_summary.md, chat_history.jsonl}  │
└──────────────────────────────────────────────────────────┘
```

---

## 10. Undecided Items

- Maximum number of turns to keep in `chat_history.jsonl` (default candidate: last 50 turns)
- Whether to include archived nodes in `recall` targets
- Whether to add a `visualize` skill (output map in Mermaid format)
- Detailed specification of automatic consistency check for `index.md`
