# /ark — Structured Executor Delegation

Formats the current task as a structured delegation prompt and sends it to the executor model.

## When to Use

Run `/ark` when you have a clear task ready to delegate. Claude will:
1. Use `Grep` to confirm key line numbers (if uncertain)
2. Format a delegation brief (≤ 200 tokens of Claude output)
3. Call the executor via MCP or your configured tool
4. Only perform final verification — no line-by-line review

## Delegation Format (Claude must follow this exactly)

```
Goal: [one sentence — what should exist after this task]

Changes:
1. path/to/file:line_range | what to change (1-2 sentences)
2. path/to/file:line_range | what to change (1-2 sentences)
...

The executor reads the above files directly.
No original code is provided — executor locates and modifies independently.

Verify: [one sentence — how to confirm correctness after execution]
```

## Rules

- **No code blocks** in the delegation prompt — use file:line references only
- **No file reading by Claude** — executor handles all file I/O
- Claude output for the entire delegation: ≤ 300 tokens
- If more than 8 change items: split into two separate `/ark` calls

## Example

```
Goal: Add a progress bar to each book item in the book list.

Changes:
1. dashboard/js/books.js:45-52 | Wrap book-item content in a position:relative div. Add an absolute-positioned inner div as background progress bar (height: 100%, gold at 15% opacity). Width = chapters/target_chapters * 100%, or 0 if no target.
2. dashboard/css/main.css:210  | Add .book-progress-bar class: position absolute, top 0, left 0, height 100%, background rgba(245,166,35,0.15), transition width 0.3s.

Executor reads both files directly.

Verify: Open dashboard, book items should show a partial gold background proportional to chapter completion.
```
