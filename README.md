# Browsable Agent Skill

This repository contains public instructions for using Browsable from agents (Claude, Codex, Cursor, etc.).

## What this gives agents

- `create_scraper`: create a new agent-based Browsable task generator
- `get_scraper_generation`: poll generation status and pull `generated_task.task_key`
- `create_scraper_and_run`: one-step create + optional poll + run path

## Use

Install this skill into your agent of choice by pointing it at:

- `https://github.com/browsable-app/create-web-scraper-skill`

Then call the Browsable MCP tools when the user asks for URL-driven scraping.
