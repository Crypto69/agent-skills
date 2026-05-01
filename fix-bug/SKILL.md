---
name: fix-bug
description: Fix bugs from docs/bugs.md. Lists unfixed bugs in priority order for selection, or accepts a bug number directly. Investigates, implements, tests, and updates the bug status.
user-invocable: true
---

# Fix Bug

Investigate and fix bugs tracked in `docs/bugs.md`.

## Process

1. **Read `docs/bugs.md`** — Parse all bug entries.

2. **Determine which bug to fix:**

   - **If the user provided a bug number** (e.g. `/fix-bug 1`, `/fix-bug BUG-003`): Find that specific bug. If it's already Fixed or Won't Fix, inform the user and stop.
   
   - **If no bug number provided**: Collect all open bugs (statuses: Reported, Confirmed, In Progress) and present them in priority order. Use AskUserQuestion to let the user pick which bug(s) to fix.

3. **Priority ordering** — When listing bugs for selection, sort by:
   1. Severity: Critical > High > Medium > Low
   2. Status: In Progress > Confirmed > Reported (prefer bugs already being worked on)
   3. Age: Older bugs first (earlier reported date)

   Display as a numbered list:
   ```
   Open bugs (prioritised):

   1. [BUG-005] 🔴 Critical — Payments failing on Stripe webhook timeout (landing)
   2. [BUG-002] 🟡 High — Video upload stalls on slow connections (web-recorder)
   3. [BUG-001] 🔴 Medium — Frame changes don't enable regenerate button (landing)

   Enter bug number(s) to fix (e.g. "1" or "1,3"), or "all" to fix everything:
   ```

4. **Investigate the bug** — Before writing any code:
   - Read the bug description, steps to reproduce, and expected vs actual behaviour
   - Identify the **area** (mobile, landing, web-recorder, processing, supabase, infrastructure) and navigate to the relevant code
   - Search for the root cause using Grep, Read, and Glob
   - Understand the surrounding code and any related functionality
   - Identify the minimal set of files that need to change

5. **Update status to In Progress** — Edit `docs/bugs.md` to change the bug's status:
   ```
   | **Status** | 🔵 In Progress |
   ```

6. **Implement the fix** — Write the minimal, correct fix:
   - Follow all coding conventions from CLAUDE.md
   - Don't introduce unnecessary changes or refactors beyond the bug fix
   - If the fix spans multiple areas, handle each area
   - Add comments only if the fix is non-obvious

7. **Verify the fix** — Run relevant tests to confirm:
   - For **landing**: `cd landing && npm test`
   - For **web-recorder**: `cd web-recorder && npm test`
   - For **processing**: `cd processing && npm test`
   - For **mobile**: `cd mobile && flutter test`
   - For **supabase**: `cd supabase/functions/tests && deno test`
   - If the bug involves UI behaviour, use the Playwright CLI skill (headed mode) to validate visually
   - If tests fail, diagnose and fix before proceeding

8. **Update status to Fixed** — Edit `docs/bugs.md` to update the bug entry:
   - Change status to `🟢 Fixed`
   - Add a **Fix details** section below the existing fields:
   ```markdown
   **Fix details:**
   - **Fixed:** {YYYY-MM-DD AEST}
   - **Root cause:** {Brief explanation of what caused the bug}
   - **Files changed:** {List of files modified}
   ```

9. **Report to the user** — Summarise:
   - What the root cause was
   - What was changed and where
   - Whether tests pass
   - Any follow-up items (e.g. "this should also be tested manually on iOS")

## Fixing multiple bugs

If the user selects multiple bugs or says "all":
- Fix them one at a time in priority order
- Update each bug's status as you go
- If one fix is blocked or too complex, skip it, note why, and move to the next
- Provide a summary at the end listing which bugs were fixed and which were skipped

## Edge cases

- If `docs/bugs.md` has no open bugs, say "No open bugs to fix. Nice work!"
- If the requested bug number doesn't exist, say "BUG-{NNN} not found in docs/bugs.md"
- If the bug is already Fixed, say "BUG-{NNN} is already marked as Fixed"
- If the bug is Won't Fix, say "BUG-{NNN} is marked as Won't Fix — reopen it first with `/add-bug` or edit docs/bugs.md"
- If you cannot determine the root cause after investigation, update the status to Confirmed (if it was Reported), describe your findings, and ask the user for guidance rather than guessing at a fix
- If the fix requires changes to infrastructure, environment variables, or third-party services (Stripe, R2, etc.) that can't be done in code, document what needs to be done manually and mark the bug as `🟡 Confirmed` with a note

## Notes

- Use AEST timezone for timestamps
- Always read the relevant code before attempting a fix — never guess
- Prefer the smallest correct fix over a large refactor
- If the fix is risky or touches payment/auth code, flag it to the user before applying
- Each bug fix should be a separate commit (per project feedback on separate commits)
