# agent-skills

A collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills I use day-to-day. Drop any of them into a project's `.claude/skills/` folder (or your global `~/.claude/skills/`) to use them.

## Skills

| Skill | What it does |
|---|---|
| [`review-my-code`](./review-my-code) | Senior-engineer review of uncommitted changes. Bugs, security, perf, missing tests, YAGNI/KISS. Designed to run automatically via a Stop hook. |
| [`add-bug`](./add-bug) | Log a bug to `docs/bugs.md`. |
| [`list-bugs`](./list-bugs) | List bugs from `docs/bugs.md` with filters. |
| [`fix-bug`](./fix-bug) | Pick a bug, fix it, test it, update its status. |
| [`add-feature`](./add-feature) | Log a feature request to `docs/features.md`. |
| [`playwright-cli`](./playwright-cli) | Automate the browser via `playwright-cli` for tests, screenshots, scraping. |
| [`excalidraw-diagram-skill`](./excalidraw-diagram-skill) | Generate Excalidraw diagrams that argue visually (not just boxes-and-arrows). |

## Install one skill (per-project)

```bash
git clone https://github.com/Crypto69/agent-skills.git /tmp/agent-skills
mkdir -p .claude/skills
cp -R /tmp/agent-skills/review-my-code .claude/skills/
```

Replace `review-my-code` with whichever skill you want. Restart Claude Code (or open a fresh session) and the skill is available — for example `/review-my-code`.

## Install all skills (global)

```bash
git clone https://github.com/Crypto69/agent-skills.git /tmp/agent-skills
mkdir -p ~/.claude/skills
cd /tmp/agent-skills
for d in */; do cp -R "$d" ~/.claude/skills/; done
```

## Auto-run `review-my-code` after every Claude turn

The flagship use case — every time Claude finishes a turn, a fresh subagent reviews the uncommitted diff before you commit it.

Add this to your project's `.claude/settings.json` (or `~/.claude/settings.json` for global):

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

How it works:

- **`SessionStart`** snapshots the working-tree state to `/tmp/cc-review-<session_id>` (HEAD commit + `git diff HEAD` + untracked files, hashed).
- **`Stop` precheck** re-hashes the tree at end-of-turn. If it matches the snapshot, exit 1 → agent is **skipped**. Plan-only sessions, exploration, research, and Q&A all cost zero tokens.
- **`Stop` agent** runs only when the tree actually changed *during this session* — pre-existing uncommitted clutter doesn't trigger reviews.

| Field | Purpose |
|---|---|
| `SessionStart` command | Snapshot working-tree hash, keyed by session id. |
| `Stop` precheck | Compare current tree to snapshot; exit 1 to skip review. |
| `type: "agent"` | Spawn a fresh subagent (separate context) to run the review. |
| `prompt` | What the subagent is asked to do. |
| `model` | Reviewer model. Sonnet is fast + cheap; switch to Opus for deeper reviews. |
| `timeout` | Seconds before the hook is killed (180s handles most diffs). |
| `statusMessage` | Shown in the Claude Code status line while reviewing. |

Requires `jq` on `PATH` (preinstalled on macOS, available via apt/brew elsewhere).

After editing settings, open `/hooks` once or restart Claude Code so the watcher picks up the new config.

See [`review-my-code/README.md`](./review-my-code/README.md) for full details.

## Uninstall

```bash
rm -rf .claude/skills/<skill-name>            # or ~/.claude/skills/<skill-name>
```

If you added the Stop hook, remove the `Stop` block from your `settings.json` too.

## License

MIT — see [LICENSE](./LICENSE).
