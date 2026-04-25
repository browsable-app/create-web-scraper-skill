# Create Web Scraper Skill

This repository contains public instructions for using Browsable from agents (Claude, Codex, Cursor, etc.).

It is designed for non-technical users: they should only need to say what URL they want scraped. The MCP tool responses carry everything the agent needs to continue.

## What this gives agents

- `create_scraper`: create a new agent-based Browsable task generator
- `get_scraper_generation`: poll generation status and pull `generated_task.task_key`
- `create_scraper_and_run`: one-step create + optional poll + run path

## Zero-knowledge onboarding flow

1. Agent calls `create_scraper_and_run` with a target URL.
2. If creation succeeds, the tool returns either:
   - `run`: ready result, or
   - `needs_user_intervention: true` with `auth_required.hosted_url` (for account/login requirements).
3. If `needs_user_intervention` is true, present that URL to the user and ask them to open it.
4. After they complete the flow, continue by polling `get_scraper_generation` with `generation_id` from `create_scraper_and_run.create_scraper.id`.

The important part is that API responses are structured and machine-readable, so the agent never has to assume user knowledge of Browsable.

## Use

Install this skill into your agent of choice by pointing it at:

- `https://github.com/browsable-app/create-web-scraper-skill`

Then call the Browsable MCP tools when the user asks for URL-driven scraping.
