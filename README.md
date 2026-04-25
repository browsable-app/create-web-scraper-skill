# Create Web Scraper Skill

This repository contains public instructions for using Browsable from agents (Claude, Codex, Cursor, etc.).

It is designed for non-technical users: they should only need to say what URL they want scraped. The MCP tool responses carry everything the agent needs to continue.

## What this gives agents

- `create_scraper`: create a new agent-based Browsable task generator
- `get_scraper_generation`: poll generation status and pull `generated_task.task_key`
- `run_task`: execute the generated task key
- `get_run`: poll run status/results until terminal

## Zero-knowledge onboarding flow

1. Call `create_scraper` with the target URL.
2. Poll `get_scraper_generation` with `generation_id` from `create_scraper.result.id` until status is `completed`.
3. Call `run_task` with `generated_task.task_key`.
4. Poll `get_run` with returned `run_id` until status is terminal (`succeeded`, `failed`, `errored`, `cancelled`).
5. If auth is required, surface `auth_required.hosted_url` and ask the user to complete it, then resume polling.

The important part is that API responses are structured and machine-readable, so the agent never has to assume user knowledge of Browsable.

## Use

### Install this skill package

```bash
npx skills add browsable-app/create-web-scraper-skill
```

Install to all supported agents:

```bash
npx skills add browsable-app/create-web-scraper-skill --all
```

### Install directly

Install this skill into your agent of choice by pointing it at:

- `https://github.com/browsable-app/create-web-scraper-skill`

Then call the Browsable MCP tools when the user asks for URL-driven scraping.
