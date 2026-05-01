# fix-bug

Pick a bug from `docs/bugs.md`, investigate, implement a fix, test it, and update the bug status.

## Usage

```
/fix-bug         # lists unfixed bugs in priority order, then prompts you to pick one
/fix-bug 3       # fixes bug #3 directly
```

After the fix is verified, the skill updates the bug's status in `docs/bugs.md`.
