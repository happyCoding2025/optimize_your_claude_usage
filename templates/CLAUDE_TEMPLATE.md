# Claude Code Global Instructions

## User
- Language preference: [your language]
- Interruption policy: Do not interrupt before task completion
- Workspace: [your workspace path]

---

## Tool Division (if using dual-AI architecture)

| Tool | Role | Use for |
|------|------|---------|
| `claude` | Thinker | Analysis, decisions, architecture, code review |
| Cheap executor | Doer | All code writing, file modification, content generation |

**Rules**:
- Code changes (new features, refactors, multi-file edits) → delegate to executor
- Content generation (docs, reports, bulk text) → delegate to executor
- **Exception**: single-line emergency fixes (typos, missing imports) → Edit tool directly

---

## Token Saving Rules (Target ratio Claude:Executor = 1:7)

### Reading Files
- **Never** `Read` an entire file larger than 10KB
- Strategy: `Grep` to locate the relevant lines → `Read` with `offset` + `limit` for only the needed section
- When delegating: instruct the executor to read the files itself — do not read on its behalf

### Delegating to Executor
- Use `file_path:line_number` to identify change locations — **never paste original code blocks**
- Format: `file:line_range | change description (1-2 sentences) | context (1 sentence if needed)`
- For multiple changes: provide a numbered list, no code expansion

### Context Management
- Remind user to `/compact` when conversation context reaches ~50%
- Suggest `/clear` after each independent task is completed

---

## Executor Delegation Format

When handing off a task to the executor model, use this template:

```
Goal: [one sentence describing the outcome]

Change list:
1. path/to/file.py:45-60 | description of change
2. path/to/other.js:120  | description of change

The executor reads the above files directly. Do not include original code.

Verification: [how to confirm correct implementation]
```

Maximum Claude output per delegation: ~200 tokens.

---

## File Reading Strategy

```
# Instead of:
Read("src/components/Dashboard.tsx")   # 800 lines = 16,000 tokens

# Do:
Grep("function Dashboard")             # → found at line 45
Read("src/components/Dashboard.tsx", offset=40, limit=60)  # 60 lines = 1,200 tokens
```

---

## Task File Management

- Keep active `TASK.md` under 50 lines
- Archive completed items to `DONE.md` after each major milestone
- Each task entry: one line summary + acceptance criteria only
