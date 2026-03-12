# File Splitting Guide

Large files are the single biggest source of wasted input tokens. This guide covers when and how to split them.

## The Problem

```
200KB single file = ~50,000 tokens

Every time you touch ANY part of this file:
- Claude reads the whole thing: 50,000 input tokens = $0.15
- Executor gets the whole thing as context
- Grep results point to this one massive file

Over 40 turns in a session: 50,000 × 40 = 2,000,000 tokens = $6.00
Just from reading one file.
```

## When to Split

Split a file when:
- It exceeds **10,000 tokens** (~40KB for code)
- It contains **more than 2 unrelated feature domains**
- You touch it **more than 3 times per session**

Don't split if:
- The file is rarely read
- Splitting would break a framework convention (e.g., Next.js page files)
- The file is auto-generated

## Splitting Strategy

### Frontend (single HTML/JS file)

Common pattern: one large HTML file with embedded CSS and JS.

**Before:**
```
index.html  (200KB, everything in one file)
```

**After:**
```
index.html          ← skeleton HTML only, imports below
js/
  app.js            ← init(), global variables, utilities
  api.js            ← fetch wrappers, URL helpers
  books.js          ← book list rendering, resume modal
  scholars.js       ← scholar management, sidebar
  stream.js         ← real-time output stream, tabs
  review.js         ← chapter review, rewrite
css/
  main.css          ← extracted from <style> tag
```

Each file: 5-20KB. Reading one feature = 1,500 tokens instead of 50,000.

### Backend (large router file)

**Before:**
```
routers/main.py  (80KB, all endpoints)
```

**After:**
```
routers/
  books.py        ← book CRUD, chapter operations
  scholars.py     ← scholar management
  generation.py   ← novel generation, SSE stream
  review.py       ← review, batch rewrite
  system.py       ← health check, logs, queue
```

### How to Instruct the Executor

When asking your executor model to split a file, give it this template:

```
Goal: Split dashboard/index.html (200KB) into feature modules.

Target structure:
  dashboard/js/app.js       ← init(), global vars (_books, _scholars, etc.)
  dashboard/js/api.js       ← getApiUrl(), all fetch() wrappers
  dashboard/js/books.js     ← renderBooks(), showContinueModal(), loadTodayUpdates()
  dashboard/js/scholars.js  ← loadScholarsIntoHanlin(), selectScholarNode()
  dashboard/js/stream.js    ← connectToJob(), createStreamTab(), addStreamItem()
  dashboard/js/review.js    ← reviewBatch(), rewriteChapter()
  dashboard/css/main.css    ← all CSS from <style> tag

Rules:
- index.html keeps only the HTML skeleton + <script src="..."> tags
- No functionality changes, only reorganization
- Maintain all existing function names and global variable names
- Execution order in index.html: api.js → app.js → feature files → init()

Executor reads index.html directly and performs the split.

Verify: Page loads without console errors, all buttons work.
```

## After Splitting: Update Your Navigation

Create a `PROJECT_MAP.md` listing key functions and their new locations:

```markdown
# Function Location Index

## Core
init()                  app.js:12
getApiUrl()             api.js:5
_books (global)         app.js:8

## Books
renderBooks()           books.js:45
showContinueModal()     books.js:180
loadTodayUpdates()      books.js:290

## Generation
connectToJob()          stream.js:15
addStreamItem()         stream.js:88
createStreamTab()       stream.js:55

## Review
reviewBatch()           review.js:20
rewriteChapter()        review.js:95
```

With this map, Claude can answer "where is renderBooks?" without reading any file.
