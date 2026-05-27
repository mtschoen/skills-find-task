# find-task

A Claude Code skill that surfaces 3–5 open tasks from across your tracked projects, matched to whatever you're in the mood for. projdash does the fetching; the skill does the picking — there is no scoring layer, taste lives in the prompt.

## When it fires

You ask Claude to pick something to work on: "find me a task", "what should I work on", "pick me something fun/quick/meaty", "something fresh to dig into", "help me find something to do".

## What it does

Reads mood signals from your request, calls `mcp__projdash__find_open_tasks` across ACTIVE projects, filters with judgment (mood match, project health, momentum), and presents a handful of candidates with one-line reasoning and a `source_file:line` pointer each. Requires the projdash MCP server.

Companion skills: `reconcile-tasks` (finds tasks already *done*) and `project-maintenance` (maintenance sweeps).

The authoritative spec is [`SKILL.md`](SKILL.md).

**Repo:** <https://github.com/mtschoen/skills-find-task>
