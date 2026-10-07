---
name: daily-work-summary
description: Produce an end-of-day summary of a user's work across every surface — workspace docs, git repos, chat sessions, cron jobs, and Jira/Confluence — grouped by project. Read-only. Use for "summarize my day", "daily standup", "EOD recap".
triggers: daily summary, summarize my day, what did i do today, daily standup, eod summary, end of day recap, project recap, daily digest, whats in flight
source: community
created_at: 2026-10-07T00:17:00Z
---

# daily-work-summary

## When to use

When the user asks for a recap of their work for a day — "summarize my day",
"daily standup", "what did I do today", "EOD summary". A future session, for any
user, loads this to assemble one coherent cross-surface digest. It is
**read-only**: it never edits a repo, doc, cron, or Jira/Confluence item.

Lead with outcomes and what moved, not an activity log. Group by project, give
concrete artifacts (PR numbers, file paths, ticket keys), and omit empty sections.

## Steps

1. **Scope the window.** Default to today in the user's local timezone (use the
   `[CURRENT DATE]` block as the clock, never training cutoff). Honor a named
   window ("yesterday", "this week"). State the window at the top of the report.

2. **Gather from every surface in parallel.** Skip a source with no signal silently.

   - **Workspace / notes docs** — rolling outcome or status docs the user keeps:
     `ls -1t <workspace>/*.md`, plus any task queue (e.g. `HEARTBEAT.md`). These
     name the user's active programs.
   - **Git repos** — commits and uncommitted changes in the window. Classify each
     repo as work vs personal by its **remote URL** (an org/work remote is in
     scope; a personal GitHub remote is out) unless the user says otherwise:
     ```bash
     for d in <project-roots>/*/; do
       [ -d "$d/.git" ] || continue
       echo "== $d =="
       git -C "$d" log --since="midnight" --oneline 2>/dev/null
       git -C "$d" status --porcelain 2>/dev/null
     done
     ```
   - **Loose deliverable files** — docs/configs the user edits outside git:
     `find <project-roots> -maxdepth 1 -type f -mtime -1 2>/dev/null`.
   - **Chat sessions** — `list_sessions(limit=20)`; read a thin-titled one in the
     window with `get_chat_session`, or `search_chat_history(query, after=<ISO>)`.
   - **Cron jobs** — `cron_list()` plus `kirocrew cron list` (shell) for jobs owned
     elsewhere. Report any that ran today or need attention. No jobs is normal.
   - **Jira + Confluence** (if Atlassian connected). A 401 "scope does not match"
     is a stale warm-pool token; the fix is human-only (Connections → Atlassian →
     Authorize → Apply & Restart). Note it, don't fail.
     ```
     searchJiraIssuesUsingJql(jql="assignee = currentUser() AND updated >= startOfDay() ORDER BY updated DESC")
     searchConfluenceUsingCql(cql="contributor = currentUser() AND lastmodified >= now('-1d') AND type = page ORDER BY lastmodified DESC")
     ```
     For one cross-tool sweep use `search(query=...)` (Rovo). **Never write** to
     Jira/Confluence from this skill.

3. **Render as an inline widget.** `<mcwidget title="Daily Summary — <date>">`,
   themed with CSS variables (never fixed colors — set every background with its
   text color so it reads in dark mode). Structure: window line → headline (1–2
   sentences on the biggest thing that moved) → by-project blocks (what moved,
   artifact, done vs in-flight) → in flight / open → scheduled jobs (only if any)
   → blocked / needs you (each with the one action needed). Keep it tight.

4. **Offer a daily cron only if asked.** Register with `cron_add` as an **agent**
   job (it reasons over mixed sources): set `cron_expr` with an explicit IANA
   `timezone`, `persistent_session=false`, `hide_in_chat=true`, and a message that
   says "Load the daily-work-summary skill and produce today's summary." Do NOT set
   `minimal_context=true` — the job needs this skill's instructions and project
   context. Confirm time + channel first, then `cron_trigger` once to smoke-test.

## Gotchas

- **Classify repos by remote, not by folder name.** Ask once which projects are
  "work" if the remotes don't make it obvious, then respect that split.
- **Don't invent activity.** A quiet day is a short summary. Report only real
  signal; omit empty sections rather than writing "nothing here".
- **Atlassian 401 is a connection state, not a skill failure.** Relay the
  human-only fix and continue with the other sources.
- **Read-only.** This skill never commits, edits, or files tickets — if the user
  wants that, it's a separate explicit action.
