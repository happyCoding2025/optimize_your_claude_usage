# Claude Code Cost Breakdown

## Pricing (claude-sonnet-4-6, as of 2025)

| Token Type | Price |
|-----------|-------|
| Input | $3.00 / million tokens |
| Output | $15.00 / million tokens |

Output costs **5× more** than input. This shapes every optimization decision.

---

## How a Session Accumulates Cost

Each API call sends the **entire conversation history** as input. As turns increase, input tokens grow:

```
Turn 1:  15,000 tokens context → $0.045 input
Turn 10: 50,000 tokens context → $0.150 input
Turn 25: 100,000 tokens context → $0.300 input
Turn 40: 150,000 tokens context → $0.450 input

Sum of all 40 turns ≈ 3.2M tokens input = $9.60
```

Plus output (~600 tokens × 40 turns = 24k tokens = $0.36).

**Total: ~$10 per heavy session.**

---

## Token Cost by Action

| Action | Tokens | Cost |
|--------|--------|------|
| Read 200KB file | 50,000 input | $0.15 |
| Read 200KB file (targeted, 80 lines) | 2,000 input | $0.006 |
| Write ARK task with code quotes | 1,500 output | $0.0225 |
| Write ARK task with file:line refs | 150 output | $0.00225 |
| Load 1,200-line TASK.md per turn | 8,000 × turns input | $0.24 per 10 turns |
| Load 50-line TASK.md per turn | 350 × turns input | $0.011 per 10 turns |
| One full session to context limit | ~3.2M input + 24k output | ~$10 |

---

## Break-Even Analysis: Dual-AI

If your executor model costs 1/7th of Claude:

```
Same task, two approaches:

Approach A: Claude does everything
  Claude output: 5,000 tokens = $0.075

Approach B: Claude delegates to executor
  Claude output: 150 tokens = $0.00225
  Executor output: 5,000 tokens × (1/7) = $0.0107
  Total: $0.01295

Savings: 83% per task
```

Break-even point: executor is worth using if it costs less than Claude × (1 - overhead_ratio).
For a 1/7 price executor, it's worth it for any task over ~50 tokens of output.

---

## Monthly Cost Scenarios

### Scenario 1: Solo developer, 3 heavy sessions/week

```python
sessions_per_week = 3
weeks_per_month = 4
turns_per_session = 40
avg_context = 80_000  # tokens, average over session lifetime
avg_output = 600      # tokens per turn

monthly_input = sessions_per_week * weeks_per_month * turns_per_session * avg_context
monthly_output = sessions_per_week * weeks_per_month * turns_per_session * avg_output

cost = monthly_input / 1e6 * 3 + monthly_output / 1e6 * 15
# = $115.20/month (unoptimized)
# = $14.40/month (with all 5 rules applied)
```

### Scenario 2: Adding a cheap executor (1/7 price)

```
Without executor: Claude handles all 5,000 output tokens/task
  Cost: 5,000 × $15/M = $0.075/task

With executor (ARK or similar):
  Claude: 150 × $15/M = $0.00225
  Executor: 5,000 × $15/7/M = $0.0107
  Total: $0.01295/task

Savings: 83% vs Claude-only
Monthly executor cost (at this rate): ~$24 for heavy use
Monthly Claude cost: drops from $115 to ~$10
Net change: $115 → $34 (70% total reduction)
```

---

## What Doesn't Save Much

- **Reducing input tokens when you must read a file**: Input is cheap. The savings from reading 80 lines vs 800 lines is $0.002. Not worth complex engineering if it slows you down.
- **Shortening system prompts under 1,000 tokens**: Negligible impact.
- **Fewer tool calls**: Tool calls have minimal overhead. Batching is good but not primarily for cost reasons.

## What Saves the Most

1. **Avoiding large file reads** (save 40,000+ tokens per read)
2. **file:line references instead of code quotes** (save 1,000+ output tokens per delegation)
3. **Proactive /compact** (save 100,000+ input tokens per session)
4. **Lean TASK.md** (save 7,500 tokens × every turn)
5. **Executor for all code writing** (save 80%+ on code-heavy tasks)
