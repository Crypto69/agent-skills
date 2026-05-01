---
name: list-bugs
description: List and summarize bugs from docs/bugs.md, with optional filtering by status, severity, or area.
user-invocable: true
---

# List Bugs

Display a summary of bugs tracked in `docs/bugs.md`.

## Process

1. **Read `docs/bugs.md`** — Parse all bug entries.

2. **Apply filters** — If the user provided arguments, filter by:
   - **Status**: `reported`, `confirmed`, `in-progress`, `fixed`, `wont-fix`
   - **Severity**: `critical`, `high`, `medium`, `low`
   - **Area**: `mobile`, `landing`, `web-recorder`, `processing`, `supabase`, `infrastructure`
   
   Arguments are matched case-insensitively. Multiple filters narrow the results (AND logic). If no arguments are given, show all **open** bugs (everything except Fixed and Won't Fix).

3. **Display as a table** — Output a markdown table with these columns:

```
| ID | Status | Severity | Area | Title | Reported |
```

4. **Show a count summary** — Below the table, show totals like:
   - "5 bugs shown (2 Critical, 1 High, 2 Medium)"
   - If filtered: "3 of 8 bugs match filter"

## Examples

- `/list-bugs` — All open bugs
- `/list-bugs critical` — Only critical severity
- `/list-bugs reported landing` — Reported bugs in the landing app
- `/list-bugs all` — Everything including Fixed and Won't Fix

## Edge cases

- If `docs/bugs.md` has no bug entries, say "No bugs tracked yet. Use `/add-bug` to log one."
- If filters match nothing, say "No bugs match that filter" and show the total count of all bugs.
