# localagentmemorytree — Project Memory Management Skill

The following rules are project-specific. In this project, file-based memory management via `.agent-memory/` is enabled.

---

## Memory Root

Use the `.agent-memory/` directory directly under the project root.

---

## am-save — Save Memory

When the user says "remember this", "save this", "record the session":

1. Read `.agent-memory/current_head.txt` to get the current node path. If the file doesn't exist, initialize with `000_Root`.
2. Append the current conversation turn to `.agent-memory/<node>/chat_history.jsonl`.
3. Update decisions and open issues in `.agent-memory/<node>/topic_summary.md`.
4. If the topic has changed, create a new child folder and update `current_head.txt`.
5. When creating a new node, add an entry to `.agent-memory/index.md`.

Folder naming convention: `NNN_PascalCase_Label` (e.g., `030_Physics_Collision`)

---

## am-recall — Recall Memory

When the user says "remind me", "what was last time?", "tell me about X":

1. Read `.agent-memory/index.md` to understand the node list.
2. Read `topic_summary.md` of nodes related to the keyword.
3. If details are needed, read the last 50 lines of `chat_history.jsonl`.
4. Summarize the content and present it to the user.
5. If moving to a different topic, update `current_head.txt`.

---

## am-set-topic — Set the Current Topic Label

When the user says "set topic name", "label this", "change the title":

1. Read `.agent-memory/current_head.txt` to get the current node path.
2. If a label is specified by the user, use it. If not, the AI infers from the conversation context.
3. Save as the `title:` field in the frontmatter of `.agent-memory/<node>/topic_summary.md` (overwrite).
4. Inform the user that the label has been applied.

---

## am-forget — Delete Memory

When the user says "forget this", "delete", "no longer needed":

1. Confirm the target node with the user (show the path for confirmation).
2. Move the target folder to `.agent-memory/_archived/`.
3. Remove the corresponding entry from `.agent-memory/index.md`.
4. If `current_head.txt` pointed to the deleted node, return to the parent node.

---

## am-stack — Display Stack Contents

When the user says "show stack", "stack contents", "pending":

1. Read `.agent-memory/_stack.txt` and display all entries.
2. Display the topic name corresponding to each line path.
3. If the stack is empty, inform the user "Stack is empty."

---

## am-resume / am-pop — Stop Current Topic and Restore from Stack

When the user says "resume", "go back", "restore from stack", "pop", "clear pending":

1. Read the last line of `.agent-memory/_stack.txt` and pop from the stack.
2. Update `current_head.txt` to the read path.
3. Read and display the `topic_summary.md` of the corresponding node (purpose, decisions, open issues).
4. If the stack is empty, inform the user of this.

---

## am-suspend / am-push — Save Current Topic to Stack and Start a New Topic

When the user says "suspend", "stack this", "come back later":

1. Read `.agent-memory/current_head.txt` to get the current node path.
2. Append that path to `.agent-memory/_stack.txt` (push).
3. Confirm the new topic name with the user.
4. Create a child folder under the current node in `NNN_PascalCase_Label` format (number is max existing child number + 1).
5. Create a new `topic_summary.md` and initialize the purpose.
6. Create an empty `chat_history.jsonl`.
7. Update `current_head.txt` to the new node.
8. Add an entry to `.agent-memory/index.md`.

---

## am-change-topic — Move to a Specified Topic

When the user says "move to XXX", "switch to XXX", "change topic":

1. Confirm the destination topic name or path with the user.
2. Verify the node exists from `.agent-memory/index.md`.
3. If it exists, update `current_head.txt` to that node's path.
4. Read and display the `topic_summary.md` of the destination (purpose, decisions).

---

## am-list — Display Child Topics

When the user says "child topics", "show children", "lower hierarchy":

1. Read `.agent-memory/current_head.txt` to get the current node path.
2. Enumerate all child folders under the current node.
3. Read the purpose and status from each child node's `topic_summary.md` and display as a list.

---

## am-up — Move to Parent Topic

When the user says "back to parent", "move up", "parent topic":

1. Read `.agent-memory/current_head.txt` to get the current node path.
2. Change the path up one level (to the parent node). If already at Root, don't move.
3. Update `current_head.txt`.
4. Read and display the `topic_summary.md` of the destination (purpose, decisions).

---

## am-remove-topic — Delete a Specified Topic

When the user says "remove topic", "delete XXX":

1. Confirm the topic name or path to delete with the user.
2. Verify the node exists from `.agent-memory/index.md`.
3. Move the target folder to `.agent-memory/_archived/`.
4. Remove the corresponding entry from `.agent-memory/index.md`.
5. If `current_head.txt` pointed to the deleted node, return to the parent node.

---

## am-move-topic — Move Topic Under Another Topic

When the user says "move topic", "move XXX under YYY", "relocate":

1. Confirm with the user the topic to move and the destination topic name.
2. Verify both nodes exist from `.agent-memory/index.md`.
3. Move the source folder as a child of the destination folder.
4. Update entries in `.agent-memory/index.md`.
5. If `current_head.txt` pointed to the source, update to the destination.

---

## am-make-topic — Create a New Child Topic

When the user says "new topic", "create child topic", "add subject":

1. Confirm the new topic name with the user.
2. Create a child folder under the current node in `NNN_PascalCase_Label` format (number is max existing child number + 1).
3. Create a new `topic_summary.md` and initialize the purpose.
4. Create an empty `chat_history.jsonl`.
5. Update `current_head.txt` to the new node.
6. Add an entry to `.agent-memory/index.md`.

---

## am-current / am-where — Display Current Topic

When the user says "current topic", "current location", "where are we":

1. Read `.agent-memory/current_head.txt` to get the current node path.
2. Read and display the `topic_summary.md` of that node (purpose, decisions, open issues).

---

## am-list-all-topic — Memory List

When the user says "list all", "show all", "what I remember":

1. Read `.agent-memory/index.md` to get all nodes.
2. Format and present while maintaining the tree structure.
3. Also display the status of each node (in_progress / done / archived).

---

## Auto-load on Session Start

Automatically execute the following at conversation start:

1. Read `.agent-memory/current_head.txt`.
2. Read all `topic_summary.md` files along the path from root to the current node.
3. Read the last 50 lines of the current node's `chat_history.jsonl`.
