---
name: linear-feedback
description: Import feedback from any source (URL, file, or pasted text) into Linear issues. Groups feedback into themes and creates issues automatically.
version: 1.0.0
allowed-tools:
  - WebFetch
  - Read
  - Bash
---

# Linear Feedback Import

One-command workflow to turn raw feedback into structured Linear issues.

---

## How to Invoke

```
/linear-feedback <input>
```

Where `<input>` is one of:
- A URL (Google Doc, web page, etc.)
- A file path (e.g. `~/Desktop/feedback.txt`)
- Pasted raw text directly after the command

---

## Step-by-Step Workflow

### Step 1 — Detect Input Type

- **URL** (starts with `http://` or `https://`) → fetch with `WebFetch`
- **File path** (starts with `/`, `~`, or `./`, or ends with known extension) → read with `Read`
- **Everything else** → treat as raw pasted text

### Step 2 — Parse and Group

Read the content and identify logical themes. Group related items together. Keep standalone one-off items as individual issues. Do NOT create artificial groupings — only group items that are genuinely related.

### Step 3 — Detect Priorities

Scan each item for priority callouts:

| Text in feedback | Linear Priority |
|---|---|
| `P1`, `urgent`, `critical`, `must` | Urgent |
| `P2`, `high priority`, `important` | High |
| `P3`, or no callout (default) | Medium |
| `P4`, `low`, `nice to have` | Low |

### Step 4 — Create Issues in the Team Backlog (Issues section)

All feedback goes into the **Unjumble team backlog** — not into any project. This keeps feedback in the "Issues" view in Linear.

**IMPORTANT:** Always run with env loaded:
```bash
set -a && source ~/.claude/.env && set +a
cd ~/.claude/skills/linear
```

**Team ID:** `0bef658d-5134-43e0-a216-c66bcce77e95`

**Create a top-level issue (standalone or group parent):**
```bash
npm run query -- 'mutation { issueCreate(input: { teamId: "0bef658d-5134-43e0-a216-c66bcce77e95", title: "<Title>", description: "<Description>", priority: <1-4> }) { success issue { id identifier url } } }'
```

**Create a sub-issue under a parent** (use the parent's UUID from the response above):
```bash
npm run query -- 'mutation { issueCreate(input: { teamId: "0bef658d-5134-43e0-a216-c66bcce77e95", title: "<Sub-title>", description: "<Detail>", priority: <1-4>, parentId: "<parent-uuid>" }) { success issue { id identifier url } } }'
```

Priority values: `1` = Urgent, `2` = High, `3` = Medium, `4` = Low

**Labels** — apply per the taxonomy (from the linear skill):
- One type label: `feature`, `bug`, `refactor`, `chore`, `spike`
- 1-2 domain labels: `backend`, `frontend`, `infrastructure`, `security`, etc.

### Step 6 — Assignment Rule

**NEVER auto-assign an issue to a person.** If assignment is needed, always ask the user first:
> "Would you like to assign UNJ-X to someone? If so, who?"

---

## After Import — Verify

```bash
set -a && source ~/.claude/.env && set +a
cd ~/.claude/skills/linear
npm run query -- "query { issues(first: 20, orderBy: createdAt) { nodes { identifier title priority project { name } state { name } } } }"
```

Show the user a summary of issues created with their identifiers and priority levels.

---

## Error Handling

- If `npm run ops -- create-issue` fails → try using a GraphQL mutation directly via `npm run query`
- If the project doesn't exist → create it first before creating issues
- If the API key isn't found → run: `set -a && source ~/.claude/.env && set +a` before any command
