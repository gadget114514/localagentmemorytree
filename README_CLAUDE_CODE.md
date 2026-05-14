# localagentmemorytree for Claude Code

**File-based memory management for Claude Code projects**

This guide covers installation, setup, and complete usage of localagentmemorytree with Claude Code. For general information, see [README.md](README.md).

## Overview

localagentmemorytree enables Claude Code to maintain persistent, hierarchical project memory across multiple sessions using pure file operations. No database or external tools required.

### Key Features

- **Automatic natural language triggers** — Say "remember this" and Claude Code routes to the right memory command
- **Hierarchical memory** — Organize topics and subtopics in a tree structure
- **Context persistence** — Claude Code auto-loads relevant memory at session start
- **Stack-based switching** — Suspend/resume topics for seamless context-switching
- **Git-friendly** — Everything stored in `.agent-memory/` as standard text files

---

## Installation

### Step 1: Copy Command Skills

Clone or download this repository, then copy the command files to your Claude Code skills directory:

```bash
# macOS / Linux
cp skills/commands/* ~/.claude/plugins/localagentmemorytree/

# Windows (PowerShell)
Copy-Item "skills/commands/*" "$env:APPDATA\Claude\plugins\localagentmemorytree\"
```

### Step 2: Verify Installation

Restart Claude Code. The following should work immediately:

1. Type: "remember this" — Should trigger `am-save`
2. Type: "what did we discuss last time" — Should trigger `am-recall`
3. Type: "create a new topic" — Should trigger `am-make-topic`

### Step 3: Initialize Memory (Optional)

On first use in a new project, Claude Code will auto-create `.agent-memory/` in your project root on the first save. Alternatively, create it manually:

```bash
mkdir .agent-memory
echo "000_Root" > .agent-memory/current_head.txt
```

---

## How It Works

### Automatic Triggering

Claude Code watches for natural language patterns and automatically invokes the correct skill:

| Natural Language | Command Triggered |
|---|---|
| "remember this", "save this session" | `am-save` |
| "remind me about X", "what did we do last time" | `am-recall` |
| "forget this", "delete this topic" | `am-forget` |
| "set topic name", "label this topic" | `am-set-topic` |
| "new topic", "create subtopic" | `am-make-topic` |
| "switch to X", "go to X" | `am-change-topic` |
| "move topic", "move XXX under YYY" | `am-move-topic` |
| "current topic", "where are we" | `am-current` |
| "show all memories", "list all" | `am-list-all-topic` |
| "suspend", "come back later" | `am-push` / `am-suspend` |
| "resume", "go back" | `am-pop` / `am-resume` |

### Session Auto-Loading

At the start of each Claude Code session:

1. Claude Code reads `current_head.txt` to find your current location
2. Loads all `topic_summary.md` files from root → current node
3. Reads the last 50 conversation turns from current node's `chat_history.jsonl`

This means your project context is automatically available without manual commands.

---

## Memory Structure

```
<project-root>/.agent-memory/
├── current_head.txt           # Current location (e.g., "000_Root/010_Features")
├── index.md                   # Index of all topics
├── _stack.txt                 # Suspend/resume stack
└── 000_Root/
    ├── topic_summary.md       # Root: project goals & decisions
    ├── 010_Features/
    │   ├── topic_summary.md   # Feature: UI improvements
    │   └── chat_history.jsonl # Conversation history
    └── 020_Architecture/
        ├── topic_summary.md   # Topic: System design
        └── chat_history.jsonl
```

### Folder Naming

Folders follow the pattern: `NNN_PascalCase_Label`

- **NNN**: Sequential 3-digit number (000, 010, 020, ...)
- **PascalCase_Label**: Human-readable topic name

Examples: `000_Root`, `010_Authentication`, `020_Database_Migration`, `030_UI_Redesign`

### File Formats

**topic_summary.md** — Topic metadata and decisions

```markdown
---
node_id: 010_Features
parent: 000_Root
status: in_progress    # in_progress | done | archived
updated: 2026-05-14T10:30:00
---

## Purpose
Implementing new UI features for the dashboard

## Decisions
- Use React hooks for state management
- Implement dark mode toggle
- Mobile-first responsive design

## Open Issues
- Define animation timings
- Finalize color palette
- Test on Safari 14
```

**chat_history.jsonl** — Conversation turns (one JSON object per line)

```jsonl
{"turn": 1, "timestamp": "2026-05-14T10:00:00", "role": "user", "content": "Let's improve the dashboard UI"}
{"turn": 2, "timestamp": "2026-05-14T10:00:05", "role": "assistant", "content": "I'll help design the new UI layout..."}
```

---

## Complete Command Reference

All 15 commands are documented below with triggers and usage examples.

### Save Memory

**Command:** `am-save`  
**Triggers:** "remember this", "save this session", "keep in memory"

Saves the current conversation to memory. Claude Code automatically updates decisions and open issues in the current topic's `topic_summary.md`.

```
User: "Remember that we decided to use React hooks"
Claude Code: Saves to current topic's chat_history.jsonl and updates topic_summary.md
```

### Recall Memory

**Command:** `am-recall`  
**Triggers:** "remind me about X", "what did we do last time", "tell me about feature X"

Searches memory by keyword and displays relevant topics, decisions, and past conversations.

```
User: "Remind me about the authentication system"
Claude Code: Loads authentication topic summary and recent chat history
```

### Delete Memory

**Command:** `am-forget`  
**Triggers:** "forget this", "delete this topic", "no longer needed"

Archives a specified topic (moves to `_archived/` folder instead of deleting).

```
User: "Forget the old implementation approach"
Claude Code: Moves topic to _archived/ and updates index.md
```

### Set Topic Label

**Command:** `am-set-topic`  
**Triggers:** "set topic name", "label this topic", "rename this topic"

Sets or updates the title of the current topic. Claude Code saves the label in the `title:` field of the current topic's `topic_summary.md` frontmatter.

```
User: "Label this as API Authentication Implementation"
Claude Code: Updates the current topic's title in topic_summary.md
```

### Create Subtopic

**Command:** `am-make-topic`  
**Triggers:** "new topic", "create subtopic", "add subject"

Creates a new child topic under the current location.

```
User: "Create a new topic for the API redesign"
Claude Code: Creates 020_API_Redesign/ under current node, makes it active
```

### Remove Topic

**Command:** `am-remove-topic`  
**Triggers:** "remove topic X", "delete XXX", "archive this area"

Archives a specific topic by path or name.

```
User: "Remove the deprecated authentication topic"
Claude Code: Archives old_auth topic, updates current location if needed
```

### Change Topic

**Command:** `am-change-topic`  
**Triggers:** "switch to X", "go to X", "move to XXX"

Moves to a different topic and loads its context.

```
User: "Switch to the authentication topic"
Claude Code: Updates current_head.txt, loads auth topic_summary.md and history
```

### Move Topic

**Command:** `am-move-topic`  
**Triggers:** "move topic", "move XXX under YYY", "relocate topic"

Relocates a topic as a child of another topic in the hierarchy. Updates the folder structure and all references in `index.md`.

```
User: "Move the login flow topic under authentication"
Claude Code: Moves login_flow folder under auth folder, updates index.md
```

### Current Topic

**Command:** `am-current` (alias: `am-where`)  
**Triggers:** "current topic", "where are we", "current location"

Displays the current topic and its context.

```
User: "Where are we right now?"
Claude Code: Shows current topic_summary.md with decisions and issues
```

### Move to Parent

**Command:** `am-up`  
**Triggers:** "go to parent", "move up", "back to parent topic"

Moves one level up in the topic hierarchy.

```
User: "Go back to the parent topic"
Claude Code: Updates location to parent, displays parent topic_summary.md
```

### List Child Topics

**Command:** `am-list`  
**Triggers:** "show subtopics", "children", "what's under this"

Lists all child topics under the current location.

```
User: "What topics are we tracking?"
Claude Code: Lists all child folders with their status and purpose
```

### List All Topics

**Command:** `am-list-all-topic`  
**Triggers:** "list all", "show memory", "all topics", "memory map"

Displays the complete topic tree with hierarchical structure.

```
User: "Show me the entire memory structure"
Claude Code: Displays index.md with tree view of all topics and statuses
```

### Suspend Topic (Push to Stack)

**Command:** `am-push` (alias: `am-suspend`)  
**Triggers:** "suspend", "stack this", "come back later", "pause this"

Saves the current topic to a stack and starts a new task without losing context.

```
User: "I need to handle an urgent bug. Let's suspend this work."
Claude Code: Pushes current topic to stack, creates new urgent_bug topic
```

### Resume Topic (Pop from Stack)

**Command:** `am-pop` (alias: `am-resume`)  
**Triggers:** "resume", "go back", "pop", "unstable", "back to what we were doing"

Restores the most recent suspended topic.

```
User: "Done with the bug fix. Let's resume."
Claude Code: Pops from stack, restores previous topic, loads its context
```

### View Stack

**Command:** `am-stack`  
**Triggers:** "show stack", "pending work", "suspended topics"

Displays all suspended topics waiting to be resumed.

```
User: "What's on the stack?"
Claude Code: Shows all pending topics in resume order
```

---

## Usage Patterns

### Pattern 1: Feature Development

```
1. User: "Create a new topic for the payment system"
   → Claude Code: Creates 020_Payment_System/

2. User: "Save our decisions about payment processing"
   → Claude Code: Records architecture decisions to topic_summary.md

3. Later session...
   → Claude Code: Auto-loads payment system context from previous session

4. User: "Remind me what we decided about payment validation"
   → Claude Code: Recalls past decisions and relevant code discussions
```

### Pattern 2: Interrupt Handling

```
1. Working on feature development
2. User: "We need to fix a critical bug. Suspend this."
   → Claude Code: Pushes feature topic to stack, creates bug_fix topic

3. User: "Done. Resume the feature work."
   → Claude Code: Pops from stack, restores feature context automatically
```

### Pattern 3: Multi-Phase Projects

```
Root/
├── 010_Requirements/        (status: done)
├── 020_Architecture/        (status: done)
├── 030_Implementation/      (status: in_progress)
│   ├── 031_Backend/        (current location)
│   └── 032_Frontend/
└── 040_Testing/            (status: pending)
```

Use `am-up` to navigate between phases, `am-list` to see what's next.

---

## Best Practices

### 1. Write Clear Topic Summaries

When creating topics, include:
- **Purpose**: What problem does this solve?
- **Decisions**: What architectural choices have you made?
- **Open Issues**: What still needs to be decided or tested?

```markdown
## Purpose
Implement real-time notifications for user actions

## Decisions
- Use WebSockets for client-server communication
- Store notifications in PostgreSQL with 30-day retention
- Emit events from backend business logic

## Open Issues
- Should we deduplicate notifications?
- How should mobile clients handle reconnection?
```

### 2. Use Meaningful Topic Names

❌ Bad: `010_Work`, `020_Stuff`  
✅ Good: `010_User_Authentication`, `020_Database_Optimization`

### 3. Organize by Feature, Not Timeline

Organize topics by what you're building, not when you worked on it:

```
✅ 010_Authentication/
   ├── 011_Login_Flow/
   └── 012_OAuth_Integration/

❌ 010_Week_1_Work/
   ├── 011_Monday_Tasks/
   └── 012_Tuesday_Tasks/
```

### 4. Use the Stack for Real Interrupts

Push/suspend only for genuine interruptions (urgent bugs, blocked work). Don't use it to "save" topic changes between regular work sessions — just switch topics with `am-change-topic`.

### 5. Regular Archives

When a topic is complete or no longer relevant, use `am-forget` to archive it. This keeps your active memory focused.

---

## Troubleshooting

### Claude Code doesn't recognize "remember this"

**Issue:** Natural language triggers aren't being detected  
**Solution:**
1. Verify skills are copied: `ls ~/.claude/plugins/localagentmemorytree/` (macOS/Linux) or `dir $env:APPDATA\Claude\plugins\localagentmemorytree\` (Windows)
2. Restart Claude Code completely
3. Try exact trigger: `/am-save` instead of natural language

### "Stack is empty" error when trying to resume

**Issue:** Trying to pop from empty stack  
**Solution:**
1. Use `am-stack` to check what's actually saved
2. Use `am-change-topic` to navigate to a different topic manually
3. If needed, recreate the topic with `am-make-topic`

### Lost context after switching topics

**Issue:** Context from previous topic doesn't carry over  
**Solution:**
1. Check `current_head.txt` to verify location
2. Use `am-current` to reload current topic context
3. Use `am-recall` to search for specific information across all topics

### .agent-memory/ folder not created

**Issue:** Directory doesn't exist after trying to save  
**Solution:**
1. Manually create: `mkdir .agent-memory`
2. Initialize: `echo "000_Root" > .agent-memory/current_head.txt`
3. Try `am-save` again

---

## Advanced: Sharing Memory Between Agents

localagentmemorytree memory is **agent-agnostic** — you can use the same `.agent-memory/` folder with both Claude Code and OpenCode.

Same project, both agents:
```bash
# In Claude Code session
"Create a new topic for authentication"

# In OpenCode session (later)
/am-recall authentication  # Works! Loads memory from Claude Code session
```

---

## Additional Resources

- **[DESIGN.md](DESIGN.md)** — Complete technical specification
- **[AGENTS.md](AGENTS.md)** — Project-specific rules and conventions
- **[README.md](README.md)** — Generic overview (OpenCode + Claude Code)

---

## Tips for Effective Memory Management

1. **Start with a clear root topic** — Document your project's high-level goals in `000_Root/topic_summary.md`

2. **Create topics proactively** — Don't wait until you're confused; create a new topic when switching areas

3. **Review past decisions** — Use `am-recall` to check what you decided before implementing similar features

4. **Archive completed work** — Keep active memory lean by archiving done topics

5. **Use descriptions liberally** — The more context you save, the better Claude Code can help in future sessions

---

## Feedback & Issues

Found a bug or have a feature request? Check [DESIGN.md § 10. Undecided Items](DESIGN.md#10-undecided-items) for known limitations.
