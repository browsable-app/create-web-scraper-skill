---
name: browsable
description: "Create and run Browsable scrapers from URLs for agent-based workflows."
---

# Browsable

## Purpose
Use this skill when an agent is asked to create a scraper from a URL, without requiring users to know Browsable internals.

## MCP-first flow

1. Ask for target URL and, if helpful, extraction goal.
2. Call `create_scraper_and_run` with `url` and optional `name`/`description`.
3. If the result has `run`, return data to the user.
4. If `needs_user_intervention: true`, read:
   - `auth_required.hosted_url`
   - `auth_required.provider`
   - `auth_required.domain`
   - `next_step` / `needs_user_intervention`
5. Ask the user to open `auth_required.hosted_url` and complete sign in/sign up.
6. After completion, poll `get_scraper_generation` using `generation_id` from `create_scraper_and_run.create_scraper.id` until status is `completed` and then call `run_task` with the returned `generated_task.task_key`.

## Tool behavior

- `create_scraper`
  - Required: `url`
  - Optional: `name`, `description`
  - Starts agent generation and returns `id` + `status_url` under `create_scraper`.
- `get_scraper_generation`
  - Required: `generation_id`
  - Returns generation lifecycle and `generated_task` when ready.
- `create_scraper_and_run`
  - Required: `url`
  - Optional: `name`, `description`, `run_input`, `run_async`, `wait_for_completion`, `max_wait_ms`, `poll_interval_ms`
  - Returns creation result and generation status.
  - If `needs_user_intervention` is true, return contains `auth_required` with `hosted_url`.
  - Use `get_scraper_generation` to continue after auth completion.

## Error handling

- `402`: show remaining credits and ask user to top up.
- `428` from other tools: return hosted auth URL and wait for website auth completion.
- `404`: invalid generation or task key; ask for corrected input.
- `awaiting_auth` in generation status or `needs_user_intervention=true`: surface `auth_required.hosted_url`, show user-friendly instructions, and pause.
- Terminal non-success statuses (`failed`, `errored`, `cancelled`) should stop the flow and return the generation payload.
