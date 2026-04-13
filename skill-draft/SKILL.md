---
name: find-task
description: Use when the user asks Claude to pick or suggest a task to work on across their tracked projects. Triggers include "find me a task", "what should I work on", "pick me something fun/quick/meaty", "something fresh to dig into", "help me find something to do". Queries projdash for open tasks across ACTIVE projects, then selects a handful matching the user's stated mood and presents them with one-line reasoning. Requires the projdash MCP server to be available.
---

# Finding a Task

When the user asks for a task to work on, your job is to surface 3-5
candidates from the open-tasks pool that match whatever they said they're in
the mood for. The projdash MCP server does the fetching; you do the picking.
There is no scoring layer — taste lives here, in the prompt, where it can
adapt freely.

## Steps

1. **Read the user's request for mood signals.** Common ones:
   - **Fun / interesting / meaty** → prefer tasks where the title involves
     design, integration, heuristics, or anything that requires thinking.
   - **Quick / low-hanging fruit / small** → prefer tasks that look bounded
     (single checkbox, clear imperative verb, no nested structure).
   - **Fresh / something new** → prefer projects you haven't touched
     recently (memory or recent_commits will tell you).
   - **No mood stated** → ask one quick clarifying question, OR present a
     mixed bag of 3 labeled by vibe.

2. **Call `mcp__projdash__find_open_tasks`.** Defaults are usually fine.
   Pass `max_per_project=10` if you want balanced fleet coverage and the
   user has one repo with a sprawling PLAN.md.

3. **Filter and pick.** Use judgment on the returned list:
   - Match against the user's stated mood.
   - Prefer projects whose `project_context` shows health: not drowning in
     uncommitted changes, not excessively stale, recent commits look
     coherent.
   - Spread picks across projects when reasonable — don't return 5 tasks
     from one repo unless the user asked for that repo.
   - `siblings_done / siblings_total` is a momentum signal: a task in a
     mostly-done section is usually more actionable than one where nothing
     nearby has been touched.

4. **Present 3-5 candidates.** Format each one:

   **<project_name> — <task title>**
   <one-line "why this one" reasoning that references the user's mood>
   `<source_file>:<line_number>`

5. **Ask which one (or none) the user wants to dig into.** When they pick,
   transition naturally — invoke `superpowers:brainstorming` if it's
   creative work, `superpowers:test-driven-development` for a bugfix,
   `superpowers:systematic-debugging` if it looks like an investigation.

## What this skill is NOT

- **Not a ranking algorithm.** There is no scoring. You are the filter.
- **Not a commitment.** The user can say "none of these, try again" with a
  different mood, and you re-pick.
- **Not a maintenance sweep.** If the user wants a maintenance pass, defer
  to the `project-maintenance` skill instead.

## Calibration notes

- Contradictory mood words ("fun but quick") → ask one clarifying question
  rather than guessing.
- `find_open_tasks` returns zero results → tell the user, don't invent
  work. Most likely cause: no ACTIVE projects with open tasks.
- Only one reasonable candidate → present that one, don't pad to 3 with
  weak picks.
- The MCP tool reads projdash's database at `~/.projdash/projdash.db`, so
  it works from any working directory. You don't need to be inside a
  project repo to invoke this skill.
- **Abandoned tasks are already filtered out at the projdash layer.**
  projdash's parser treats task titles wrapped in `~~...~~` (markdown
  strikethrough) as abandoned and excludes them from `find_open_tasks`
  results. If an abandoned task somehow shows up in your candidate pool,
  that's a projdash bug — surface it rather than suggesting the task.
  Companion dual: `reconcile-tasks` skill lets the user cross out tasks
  as abandoned after a "what's actually done?" audit.
