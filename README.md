# Public Prompts

Markdown prompts for AI coding agents — **Cursor**, **Claude Code**, **Codex**, **ChatGPT**, and similar tools.

Each file is self-contained. Copy it into a chat, wire it as a slash command, or adapt it into an agent skill in your own repo.

## Available prompts

| Prompt | Summary |
|--------|---------|
| [**another-pass**](prompts/general/another-pass.md) | Run a focused second pass on the agent’s last task: requirements fit, edge cases, verification, and polish — without starting over. |
| [**ship-chat**](prompts/general/ship-chat.md) | Ship session work to GitHub: commit scope, branch/worktree strategy, open a PR, and optionally wait on review, CI, and squash merge. |
| [**plan-review**](prompts/general/plan-review.md) | Revise a `.plan.md` in a loop until **READY_FOR_BUILD**: working copy, optional second-opinion reviewer, unified-diff patches. |

All prompts: [`prompts/general/`](prompts/general/)

## How to use

1. Open the prompt file you want.
2. Paste the content into your agent (or point your tool at the file if it supports path references).
3. Use a **trigger phrase** when the prompt lists one — e.g. “another pass”, “ship this”, “review this plan”, “push and open a PR”.

These prompts assume the agent already has context from the work you are continuing.

## Contributing

Contributions welcome via pull request.

- Add one prompt per file under `prompts/general/`
- Use **kebab-case** filenames (`my-workflow.md`)
- Include a **Use when:** line and optional trigger phrases at the top
- Do not commit secrets, credentials, or private project data

File template: [`prompts/README.md`](prompts/README.md)

## License

[MIT](LICENSE)
