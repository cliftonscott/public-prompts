# Optional: MCP second opinion for plan review

Use this with **[plan-review](../prompts/general/plan-review.md)** when the orchestrator runs with **`--mcp`**. A separate MCP server exposes a **`review_plan`** tool; Cursor (or another MCP client) calls it while the primary agent applies revisions.

This repo documents the **contract** only. You must supply or build an MCP server that implements it.

## What you need

1. An MCP server (stdio is typical for Cursor) with a **`review_plan`** tool
2. Cursor (or another client) configured to load that server — credentials via **IDE env**, not committed files
3. The orchestrator agent following [plan-review.md](../prompts/general/plan-review.md)

## Tool contract (recommended)

Your `review_plan` tool should accept:

| Input | Type | Required | Notes |
|-------|------|----------|-------|
| `planFilePath` | string | yes | Absolute path to the **working plan** file |
| `iterationNumber` | number | yes | Current iteration (1-based) |
| `maxIterations` | number | yes | Cap from `--max-iterations` |
| `previousIterationSummary` | string | no | Short summary when iteration > 1 |
| `effort` | string | no | Optional reasoning depth if your backend supports it |

Prefer **file path** over inline plan content so large plans are not duplicated in chat.

Return JSON (field names can match your server; the orchestrator needs):

| Output | Purpose |
|--------|---------|
| `status` or equivalent | `READY_FOR_BUILD` or `NEEDS_REVISION` |
| `feedback` | Human-readable review summary |
| `cursorPatch` or `patch` | Optional unified diff for `patch -p0` |
| `cursorInstructions` or `instructions` | Fallback when patch is impractical |
| `retryable` | Optional hint if the call failed transiently |

The reviewer must follow the same **VERDICT** semantics as the [Reviewer output contract](../prompts/general/plan-review.md#reviewer-output-contract) — either embed that contract in the MCP system prompt or map MCP JSON to the orchestrator’s parser.

## Cursor MCP configuration (pattern)

Add a server entry in your **user- or project-level** MCP config (path depends on your editor). Example shape only — **do not commit real secrets**:

```json
{
  "mcpServers": {
    "plan-review": {
      "command": "node",
      "args": ["/absolute/path/to/your-mcp-server/dist/index.js"],
      "env": {}
    }
  }
}
```

Set API keys through:

- Cursor **Settings → MCP** environment variables, or
- Your shell profile (for local stdio servers)

**Never** put API keys in this public repo, in `mcp.json` committed to git, or in plan markdown.

## Orchestrator flow (`--mcp`)

1. Copy plan to working file (`WORKING_PLAN`).
2. Call `review_plan` with `planFilePath = WORKING_PLAN` and iteration metadata.
3. On `NEEDS_REVISION`, apply patch or instructions to **working file only**.
4. Increment iteration; repeat until `READY_FOR_BUILD` or cap.
5. Copy working file back to `PLAN_FILE`.

Use the MCP server name your Cursor session exposes (often prefixed, e.g. `user-…` or `project-…` — check the tool list in the IDE).

## Building your own MCP server

Minimal approach:

1. Use the [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) or [Python SDK](https://github.com/modelcontextprotocol/python-sdk).
2. Register one tool: `review_plan`.
3. Read `planFilePath` from disk inside the server process.
4. Call your model with a system prompt that includes the reviewer output contract.
5. Return structured JSON; optionally also write a markdown report locally (gitignored).

Publish your server in its **own repository** if you want others to install it; link it from your fork of [`ai-prompts`](https://github.com/cliftonscott/ai-prompts) README if desired.

## Safety checklist

- MCP server reads plans read-only; does not modify product source during review
- No secrets in tool arguments or returned feedback
- Do not log full plan contents to shared telemetry if plans may contain sensitive design notes
- Orchestrator still owns all file writes

## Without an MCP server

Use plan-review **default** (same-session) or **`--codex`** ([setup-codex-cli.md](setup-codex-cli.md)), or paste the reviewer contract into a **separate** ChatGPT/Claude chat manually (`--second-opinion` generic path).
