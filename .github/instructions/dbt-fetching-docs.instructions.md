---
applyTo: "**"
---

# Fetching dbt Docs

## Overview

dbt docs have LLM-friendly URLs. Always append `.md` to get clean markdown instead of HTML.

## URL Pattern

| Browser URL | LLM-friendly URL |
|-------------|------------------|
| `https://docs.getdbt.com/docs/path/to/page` | `https://docs.getdbt.com/docs/path/to/page.md` |

## Quick Reference

| Resource | URL | Use Case |
|----------|-----|----------|
| Single page | Add `.md` to any docs URL | Fetch specific documentation |
| Page index | `https://docs.getdbt.com/llms.txt` | Find all available pages |
| Full docs | `https://docs.getdbt.com/llms-full.txt` | Search across all docs |

## Finding Pages

1. **Search the index first**: Fetch `https://docs.getdbt.com/llms.txt` and search titles
2. **Only if needed**: Search `https://docs.getdbt.com/llms-full.txt` for full content

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Fetching HTML URL without `.md` | Always append `.md` to docs URLs |
| Searching full docs first | Search `llms.txt` index first |
| Guessing page paths | Use `llms.txt` index to find correct paths |

> Source: [dbt-labs/dbt-agent-skills](https://github.com/dbt-labs/dbt-agent-skills) — `fetching-dbt-docs`
