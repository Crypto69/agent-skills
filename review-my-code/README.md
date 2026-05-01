# review-my-code

Critically review uncommitted changes as a senior software engineer. Catches bugs, performance issues, security vulnerabilities, code duplication, missing tests, and YAGNI/KISS violations. For Vue 3, flags opportunities for reusable composables.

The skill is **read-only** — it reports findings, it does not fix code.

## What it reviews

It runs `git status --short` and `git diff HEAD`, so it catches **staged + unstaged + untracked** changes — i.e. everything Claude just wrote that you haven't committed yet.

## Manual usage

```
/review-my-code                      # review all uncommitted changes
/review-my-code src/foo.ts src/bar.ts  # scope to specific files
```

## Automatic usage — run after every Claude turn

Add a `Stop` hook to your project's `.claude/settings.json` (or `~/.claude/settings.json` for all projects):

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "sid=$(jq -r .session_id); { git rev-parse HEAD; git diff HEAD; git status --porcelain; } 2>/dev/null | shasum > \"/tmp/cc-review-$sid\""
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "sid=$(jq -r .session_id); cur=$({ git rev-parse HEAD; git diff HEAD; git status --porcelain; } 2>/dev/null | shasum); saved=$(cat \"/tmp/cc-review-$sid\" 2>/dev/null); [ \"$cur\" != \"$saved\" ] || exit 1"
          },
          {
            "type": "agent",
            "prompt": "Invoke the review-my-code skill on the git diff of changes made in this session. Report findings concisely.",
            "model": "claude-sonnet-4-6",
            "timeout": 180,
            "statusMessage": "Reviewing changes..."
          }
        ]
      }
    ]
  }
}
```

The hook is **session-aware** so it only reviews what *this* session actually changed:

- `SessionStart` snapshots the working tree (HEAD + `git diff HEAD` + `git status --porcelain`) to `/tmp/cc-review-<session_id>`.
- The Stop precheck re-hashes the tree at end-of-turn and compares to the snapshot. If unchanged, it exits 1 and the agent is **skipped** — no tokens spent.
- The agent only runs when this session genuinely modified the tree. Plan mode, exploration, Q&A, and pre-existing uncommitted clutter are all ignored.

What each field does:

| Field | Purpose |
|---|---|
| `SessionStart` command | Snapshot tree state at session start (keyed by `session_id`). |
| `Stop` precheck | Compare current tree to snapshot; exit 1 if unchanged → skip review. |
| `type: "agent"` | Spawn a fresh subagent (with its own context window) to do the review. |
| `prompt` | What to ask the subagent. |
| `model` | Model the subagent uses. Sonnet is fast + cheap; swap for Opus if you want deeper reviews. |
| `timeout` | Seconds before the hook is killed. 180s comfortably covers most diffs. |
| `statusMessage` | Shown in the Claude Code status line while the hook runs. |

Requires `jq` (preinstalled on macOS; `apt install jq` / `brew install jq` elsewhere).

After saving, open `/hooks` once or restart Claude Code — the settings watcher only reloads on session start or via `/hooks`.

Once installed, every time Claude finishes a turn with uncommitted changes, a fresh agent reviews the diff and surfaces issues before you commit.

## Output

Findings are grouped by severity:

- **Critical** — bugs, security holes, broken behaviour
- **Warnings** — likely issues, missing tests, performance smells
- **Suggestions** — duplication, YAGNI/KISS, refactor opportunities

Followed by a 1–3 sentence summary on whether the change is merge-ready.
