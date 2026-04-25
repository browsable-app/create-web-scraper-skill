---
name: browsable
description: "Create and run Browsable scrapers from URLs for agent-based workflows."
---

# Browsable

## Purpose
Use this skill when an agent is asked to create a scraper from a URL, without requiring users to know Browsable internals.

## MCP setup

Before this skill can run tools, connect to the remote MCP server:

- Server URL: `https://mcp.browsable.app`
- Add with Codex: `codex mcp add browsable --url https://mcp.browsable.app`
- Authorize with Codex: `codex mcp login browsable --scopes tasks.read,tasks.run,runs.read,task_generations.write,billing.read,keys.manage`
- Add with Claude Code: `claude mcp add --transport http browsable https://mcp.browsable.app`
- Add with Cursor config:

```json
{
  "mcpServers": {
    "browsable": {
      "url": "https://mcp.browsable.app"
    }
  }
}
```

- Add with VS Code config:

```json
{
  "servers": {
    "browsable": {
      "type": "http",
      "url": "https://mcp.browsable.app"
    }
  }
}
```

## MCP-first flow

1. Ask for target URL and, if helpful, extraction goal.
2. Call `create_scraper` with `url` and optional `name`/`description`.
3. Poll `get_scraper_generation` using `generation_id` from `create_scraper.result.id` until status is `completed`.
4. Once complete, extract `generated_task.task_key` and call `run_task`.
5. Poll `get_run` with the returned `run_id` until run status is terminal.
6. If any generation or run response indicates auth is required, surface the `hosted_url` and ask the user to complete it, then continue polling.

## Tool behavior

- `create_scraper`
  - Required: `url`
  - Optional: `name`, `description`
  - Starts agent generation and returns `id` + `status_url` under `create_scraper`.
- `get_scraper_generation`
  - Required: `generation_id`
  - Returns generation lifecycle and `generated_task` when ready.
- `run_task`
  - Required: `task_key`
  - Optional: `async` (if false, run might still return `run_id` and non-terminal status to poll) and `input`
  - Run returns `run_id`, `run_status`, and `status_url`/`result`; always poll `get_run` until terminal.
- `get_run`
  - Required: `run_id`
  - Poll this for `run_status` and final `result`.

## Error handling

- `402`: show remaining credits and ask user to top up.
- `428` from other tools: return hosted auth URL and wait for website auth completion.
- `404`: invalid generation or task key; ask for corrected input.
- `awaiting_auth` in generation status or run status: surface `auth_required.hosted_url`, show user-friendly instructions, and pause.
- Terminal non-success statuses (`failed`, `errored`, `cancelled`) should stop the flow and return the generation payload.
