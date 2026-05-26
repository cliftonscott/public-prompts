# Public Prompts

Markdown prompts for AI coding agents — **Cursor**, **Claude Code**, **Codex**, **ChatGPT**, and similar tools.

Each file is self-contained. Copy it into a chat, wire it as a slash command, or adapt it into an agent skill in your own repo.

## Available prompts

| Prompt | Summary |
|--------|---------|
| [**another-pass**](prompts/general/another-pass.md) | Run a focused second pass on the agent’s last task: requirements fit, edge cases, verification, and polish — without starting over. |
| [**ship-chat**](prompts/general/ship-chat.md) | Ship session work to GitHub: commit scope, branch/worktree strategy, open a PR, and optionally wait on review, CI, and squash merge. |
| [**plan-review**](prompts/general/plan-review.md) | Revise a `.plan.md` in a loop until **READY_FOR_BUILD**: working copy, optional second-opinion reviewer, unified-diff patches. |
| [**wwyd**](prompts/general/wwyd.md) | **What would you do?** — one clear recommendation and why; `--do` to apply it. |

### Workflow skills (inspect → change → review)

Index: [**workflow-skills**](prompts/general/workflow-skills.md) · Typical chain: **scout → patch → prove → critique**

| Prompt | Summary |
|--------|---------|
| [**scout**](prompts/general/scout.md) | Inspect before edit; smallest safe plan (read-only) |
| [**patch**](prompts/general/patch.md) | Minimal implementation change |
| [**prove**](prompts/general/prove.md) | Run checks; prove working or fix until proven |
| [**critique**](prompts/general/critique.md) | Senior review with severities (no edits unless asked) |
| [**e2e**](prompts/general/e2e.md) | Trace UI → API → backend → data → tests |
| [**brief**](prompts/general/brief.md) | Shortest useful answer |
| [**ooda**](prompts/general/ooda.md) | Observe → Orient → Decide → Act loops |

**Optional plan-review setup** (Codex CLI, MCP): [`docs/`](docs/README.md)

All prompts: [`prompts/general/`](prompts/general/)

## How to use

1. Open the prompt file you want.
2. Paste the content into your agent (or point your tool at the file if it supports path references).
3. Use a **trigger phrase** when the prompt lists one — e.g. “another pass”, “what would you do”, “ship this”, “review this plan”.

These prompts assume the agent already has context from the work you are continuing.

## Contributing

Contributions welcome via pull request.

- Add one prompt per file under `prompts/general/`
- Use **kebab-case** filenames (`my-workflow.md`)
- Include a **Use when:** line and optional trigger phrases at the top
- Do not commit secrets, credentials, API keys, or private project data
- Optional tooling guides live under [`docs/`](docs/) — configure auth in your IDE or environment only

File template: [`prompts/README.md`](prompts/README.md)

## License

[MIT](LICENSE)
