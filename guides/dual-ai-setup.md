# Dual-AI Workflow Setup

This guide explains how to set up and maintain a Claude (thinker) + cheap executor (doer) workflow.

## Concept

```
You ──────────────────────────────────────────────────── You
 │                                                         ▲
 │ describe goal                              review result │
 ▼                                                         │
Claude                                               Claude
 │                                                         ▲
 │ 150-token spec                       execution report │
 ▼                                                         │
Executor Model ──────────── reads files, writes code ──────┘
(1/7 price)
```

Claude never writes implementation code. The executor never makes architectural decisions.

---

## Compatible Executor Models

Any model accessible via API that can:
- Read files from disk (or accept file paths to read)
- Write/edit files
- Run terminal commands

Examples:
- **Volcengine ARK** (ark-code-latest) — ~1/7 Claude price, Chinese-market focused
- **Gemini Flash** — very cheap, strong coding ability
- **GPT-4o-mini** — cheap, good for structured edits
- **Local models via Ollama** — near-zero cost, quality tradeoff

The strategies in this repo work regardless of which executor you use.

---

## Integration Options

### Option A: MCP Server (recommended for Claude Code)

Create a MCP server that wraps your executor's API. Claude Code calls it via tool use.

```python
# Minimal MCP server structure
# ~/.claude/mcp/executor/server.py

from mcp.server import Server
import anthropic  # or your executor's SDK

server = Server("executor")

@server.tool()
async def execute_task(prompt: str) -> str:
    """Send a task to the cheap executor model."""
    # Call your executor API here
    response = your_executor_client.chat(prompt)
    return response.content

if __name__ == "__main__":
    server.run()
```

Register it:
```bash
claude mcp add executor python ~/.claude/mcp/executor/server.py
```

### Option B: Shell Script Wrapper

If MCP is too complex, a shell script that Claude calls via Bash tool:

```bash
#!/bin/bash
# ~/bin/ark-execute
# Usage: ark-execute "task description"

curl -s "YOUR_EXECUTOR_API_ENDPOINT" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d "{\"prompt\": \"$1\"}" | jq -r '.content'
```

### Option C: Separate Terminal Session

Run Claude Code in one terminal, your executor in another. Claude gives you the formatted spec (via `/ark` command), you paste it to the executor terminal.

Low tech, but costs nothing to set up and still saves 80% of Claude output tokens.

---

## The Delegation Protocol

Every delegation from Claude to executor follows this structure:

```
[TASK BRIEF]
Goal: <one sentence>

Files to modify:
- path/to/file.py:45-80     | what to change
- path/to/other.js:120-135  | what to change

The executor reads these files directly.
No original code is included in this brief.

Commit message: feat: <description>
Verify: <how to test>
```

**Why no original code?**
- Executor reads the file itself (using its own tokens at 1/7 price)
- Claude avoids spending 1,000+ expensive output tokens quoting code
- Executor sees the latest version, not a potentially-stale copy

---

## Work Division Reference

| Task Type | Claude | Executor |
|-----------|--------|----------|
| Understand a bug | ✅ | ❌ |
| Decide on architecture | ✅ | ❌ |
| Write a new feature | ❌ | ✅ |
| Refactor existing code | ❌ | ✅ |
| Review executor's output | ✅ | ❌ |
| Generate bulk content | ❌ | ✅ |
| One-line typo fix | ✅ (Edit tool) | ❌ |
| Multi-file changes | ❌ | ✅ |
| Explain what code does | ✅ | ❌ |
| Write tests | ❌ | ✅ |

---

## Session Pattern

A well-structured session minimizes Claude output:

```
1. Claude reads TASK.md (50 lines = 400 tokens input) ← cheap
2. Claude reads key files with Grep + targeted Read   ← cheap
3. Claude makes decision, writes /ark spec             ← 150 tokens output
4. Executor implements, commits                        ← executor's cost
5. Claude verifies (reads diff or runs quick check)   ← cheap
6. Repeat for next task
7. /compact when context reaches 50%
8. /clear after session
```

Total Claude output per task: ~150-300 tokens
Executor output per task: ~2,000-8,000 tokens

Ratio: 1:13 to 1:53 (token count), 1:2 to 1:7 (dollar cost, accounting for price difference)

---

## Common Pitfalls

**Pitfall 1: Claude "helpfully" reads the file first**
- Symptom: Claude says "Let me read the file to understand the context..."
- Fix: Add to CLAUDE.md — "Never read a file before delegating. Give the executor the file path and let it read."

**Pitfall 2: Executor needs more context than expected**
- Symptom: Executor makes wrong changes because it doesn't understand the goal
- Fix: Add one context sentence to the spec. Still cheaper than Claude reading the file.

**Pitfall 3: Claude reviews every line of executor output**
- Symptom: Claude pastes executor's 200-line output and comments on each line
- Fix: Claude only checks: did the file change? does it run? is the specific feature working?

**Pitfall 4: Session grows too long before /compact**
- Symptom: context > 80%, each turn costs $0.50+ in input alone
- Fix: Set a personal rule — if you've done more than 3 tasks since last /compact, trigger it.
