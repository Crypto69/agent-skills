---
name: add-feature
description: Add a feature request to docs/features.md with description, priority, and implementation status. Use when the user wants to log a feature idea or enhancement.
user-invocable: true
---

# Add Feature Request

Log a feature request to `docs/features.md` with structured metadata for tracking and implementation.

## Process

1. **Get feature details** — The user will describe the feature either as arguments to the command or in their message. If no description is provided, ask for one using AskUserQuestion.

2. **Read the current file** — Read `docs/features.md` to understand the current content and find the right place to append.

3. **Generate the entry** — Create a new feature entry using this format:

```markdown
### FEAT-{NNN}: {Short title}

| Field | Value |
|-------|-------|
| **Status** | Backlog |
| **Added** | {YYYY-MM-DD HH:MM AEST} |
| **Priority** | {High / Medium / Low / Nice-to-have} |
| **Area** | {mobile / landing / web-recorder / processing / supabase / infrastructure / cross-cutting} |

**Description:**
{Detailed description of the feature}

**Notes:**
{Any implementation notes, constraints, or related context — otherwise "None"}

---
```

4. **Assign the feature number** — Look at existing entries in `docs/features.md` and increment from the highest `FEAT-NNN` number found. If no entries exist, start at `FEAT-001`.

5. **Determine priority** — If the user doesn't specify priority, infer from the description:
   - **High**: Revenue-impacting, user-blocking, or frequently requested
   - **Medium**: Meaningful UX improvement or operational efficiency gain
   - **Low**: Minor enhancement, polish, or edge-case improvement
   - **Nice-to-have**: Exploratory idea, no urgency

6. **Determine area** — Infer from the description which part of the system is affected. Use `cross-cutting` if it spans multiple areas. If unclear, ask.

7. **Append the entry** — Add the new feature entry to the end of `docs/features.md`.

8. **Confirm** — Tell the user the feature has been logged with its ID, e.g. "Logged as FEAT-003 (Medium, landing)".

9. **Do NOT implement** — After logging the feature, do NOT start implementing it. Ask the user if they would like you to implement it now. Only proceed with implementation if the user explicitly confirms.

## Status lifecycle

Only `Backlog` is used by this skill. Other statuses are for reference when features are triaged or implemented later:

| Status | Meaning |
|--------|---------|
| Backlog | Logged, not yet planned |
| Planned | Accepted and scheduled for implementation |
| In Progress | Actively being built |
| Done | Implemented and verified |
| Won't Do | Declined or out of scope |

## Notes

- Use AEST timezone for timestamps (the user is in Australia)
- Keep the short title under 60 characters
- If the user provides multiple features in one message, create a separate entry for each
