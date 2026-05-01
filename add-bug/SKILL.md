---
name: add-bug
description: Add a bug report to docs/bugs.md with timestamp, description, and status tracking. Use when the user wants to log a bug they've found.
user-invocable: true
---

# Add Bug Report

Log a bug to `docs/bugs.md` with structured metadata for future tracking and automated fixing.

## Process

1. **Get bug details** — The user will describe the bug either as arguments to the command or in their message. If no description is provided, ask for one using AskUserQuestion.

2. **Read the current file** — Read `docs/bugs.md` to understand the current content and find the right place to append.

3. **Generate the entry** — Create a new bug entry using this format:

```markdown
### BUG-{NNN}: {Short title}

| Field | Value |
|-------|-------|
| **Status** | 🔴 Reported |
| **Reported** | {YYYY-MM-DD HH:MM AEST} |
| **Severity** | {Critical / High / Medium / Low} |
| **Area** | {mobile / landing / web-recorder / processing / supabase / infrastructure} |

**Description:**
{Detailed description of the bug}

**Steps to reproduce:**
{Steps if provided, otherwise "Not yet documented"}

**Expected vs actual:**
{If provided, otherwise "Not yet documented"}

---
```

4. **Assign the bug number** — Look at existing entries in `docs/bugs.md` and increment from the highest `BUG-NNN` number found. If no entries exist, start at `BUG-001`.

5. **Determine severity** — If the user doesn't specify severity, infer from the description:
   - **Critical**: Data loss, security issue, payments broken, app crash
   - **High**: Core feature broken, blocking users
   - **Medium**: Feature partially broken, workaround exists
   - **Low**: Cosmetic, minor UX issue, edge case

6. **Determine area** — Infer from the bug description which part of the system is affected. If unclear, ask.

7. **Append the entry** — Add the new bug entry to the end of `docs/bugs.md`.

8. **Confirm** — Tell the user the bug has been logged with its ID, e.g. "Logged as BUG-003 (Medium, landing)".

## Status lifecycle

These are the valid statuses — only `Reported` is used by this skill. Other statuses are for reference when bugs are triaged or fixed later:

| Status | Meaning |
|--------|---------|
| 🔴 Reported | Newly logged, not yet investigated |
| 🟡 Confirmed | Reproduced and validated |
| 🔵 In Progress | Actively being fixed |
| 🟢 Fixed | Fix applied and verified |
| ⚪ Won't Fix | Intentional or out of scope |

## Notes

- Use AEST timezone for timestamps (the user is in Australia)
- Keep the short title under 60 characters
- If the user provides multiple bugs in one message, create a separate entry for each
