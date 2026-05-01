# playwright-cli

Automates browser interactions via the `playwright-cli` tool — for testing, form filling, screenshots, and data extraction.

## Requirements

- [`playwright-cli`](https://playwright.dev/) installed and on `PATH`.
- The skill declares `allowed-tools: Bash(playwright-cli:*)` in its frontmatter — Claude Code will prompt for permission on first use, or you can pre-allow it in `.claude/settings.json`.

## Usage

The skill is auto-invoked when you ask Claude to interact with a browser. Examples:

- "Open localhost:5173 and take a screenshot of the login page."
- "Fill in the signup form on staging and submit it."
- "Extract all the headings from https://example.com."

## Notes

- Prefer `--headed` mode when validating UI work so you can see what the agent is doing.
- See `references/` for command patterns the skill draws on.
