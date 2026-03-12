# Optimize Your Claude Code Usage

**Cut your Claude Code costs by 80%+ using a dual-AI architecture and disciplined token management.**

```
╔══════════════════════════════════════╗
║  Before:  $80–120 / month            ║
║  After:   $26 / month                ║
║  Savings: 70%+  •  same velocity     ║
╚══════════════════════════════════════╝
```

> This repository contains battle-tested strategies for reducing Claude Code costs from $80+/month to under $30/month (within Pro subscription), while maintaining or improving development velocity.

---

## The Problem

Claude Code is powerful but expensive if used naively:

- A single heavy development session (40 turns, large codebase) can cost **$10+**
- Reading a large file (200KB) costs **50,000 input tokens** every time
- Writing instructions with quoted code blocks costs **1,000-2,000 output tokens** per task
- Output tokens cost **5× more** than input tokens ($15 vs $3 per million)

## The Solution: Dual-AI Architecture + Disciplined Delegation

Use Claude for **thinking** (expensive but necessary), and a cheaper execution model for **doing** (code writing, file modification, content generation).

```
Claude (Thinker)          Cheap Model (Executor)
─────────────────         ────────────────────────
Analyze problems          Write all code
Make decisions            Modify files
Review results            Generate content
Architecture design       Batch refactoring
↓                         ↑
Brief spec (100 tokens) → Execute (5,000 tokens at 1/7 price)
```

**Result**: Claude:Executor token ratio of **1:7 or better**, total cost drops from **$80-120/month → $26/month**.

---

## Quick Start

### 1. Install the CLAUDE.md rules

Copy `templates/CLAUDE_TEMPLATE.md` content into your `~/.claude/CLAUDE.md`:

```bash
cat templates/CLAUDE_TEMPLATE.md >> ~/.claude/CLAUDE.md
```

### 2. Install the /ark command

```bash
mkdir -p ~/.claude/commands
cp commands/ark.md ~/.claude/commands/
```

### 3. Apply the 5 core rules (see below)

---

## The 5 Core Rules

### Rule 1: Grep First, Read Targeted

```
❌ Read("large_file.js")           → 50,000 input tokens
✅ Grep("functionName")            → find line 1200
   Read(offset=1190, limit=80)    → 2,400 input tokens

Savings: 96% reduction in input tokens
```

### Rule 2: file:line References, No Code Quoting

```
❌ Expensive (1,200 output tokens):
"In index.html line 1247, find this code:
  <div class="item">
    ...(50 lines of original code)...
  </div>
Change it to:
  ...(50 lines of new code)..."

✅ Efficient (120 output tokens):
"dashboard/js/books.js:47-52
Add position:relative to book-item div.
Add inner progress bar div (absolute, semi-transparent gold).
Progress = chapters/target_chapters.
Executor reads the file itself."
```

### Rule 3: Let the Executor Read Files

When delegating a task, instruct the executor model to read the relevant files itself. This transfers file-reading token costs to the cheaper model.

```
❌ Claude reads file → summarizes for executor → executor modifies
✅ Claude gives file:line refs → executor reads + modifies directly
```

### Rule 4: Keep TASK.md Lean

Archive completed tasks to `DONE.md`. Keep active task file under 50 lines.

```
50 lines × 40 turns = 2,000 tokens/session saved
vs
1,200 lines × 40 turns = 48,000 tokens/session wasted
```

### Rule 5: Proactive /compact

Trigger `/compact` when context reaches ~50%. Don't wait for forced compression.

```
Context at 50% → /compact → reset to ~15k tokens
Context at 100% → forced compact → you already paid for the full 200k
```

---

## Cost Comparison

| Scenario | Claude/month | Executor/month | Total |
|----------|-------------|----------------|-------|
| Naive usage | $80-120 | $0 | $80-120 |
| Dual-AI, unoptimized | $40 | $24 | $64 |
| Dual-AI + all 5 rules | **$2** | **$24** | **$26** |

---

## File Structure

```
optimize_your_claude_usage/
├── README.md                    ← This file
├── templates/
│   ├── CLAUDE_TEMPLATE.md       ← Drop-in CLAUDE.md rules (no personal info)
│   └── TASK_TEMPLATE.md         ← Task file template
├── commands/
│   └── ark.md                   ← /ark delegation command
└── guides/
    ├── cost-breakdown.md        ← Detailed token cost analysis
    ├── file-splitting.md        ← How to split large files
    └── dual-ai-setup.md         ← Setting up the dual-AI workflow
```

---

## Token Cost Calculator

Estimate your potential savings:

```python
# Monthly cost estimate
sessions_per_week = 3
turns_per_session = 40
avg_context_tokens = 80_000   # grows during session
avg_output_tokens = 600       # per turn

input_per_month = sessions_per_week * 4 * turns_per_session * avg_context_tokens
output_per_month = sessions_per_week * 4 * turns_per_session * avg_output_tokens

cost_input = input_per_month / 1_000_000 * 3    # $3/MTok
cost_output = output_per_month / 1_000_000 * 15  # $15/MTok
total = cost_input + cost_output

print(f"Estimated monthly cost: ${total:.2f}")
# Typical output: $115.20 unoptimized → $14.40 optimized
```

---

## Contributing

Strategies that work for your stack are welcome. Please include:
- The problem (what was expensive)
- The solution (what reduced cost)
- Rough token savings estimate

---

## License

MIT
