# SLSTech — KiroCrew Skills

Reusable [KiroCrew](https://kirocrew.ai) agent skills.

## daily-work-summary

Produces an end-of-day summary of your work across every surface — workspace docs,
git repos, chat sessions, cron jobs, and Jira/Confluence — grouped by project.
Read-only (never edits a repo, doc, cron, or ticket).

Triggers: `summarize my day`, `daily standup`, `what did I do today`, `EOD summary`.

### Install

```bash
git clone https://github.com/WGU-edu/SLSTech.git
cp -r SLSTech/skills/daily-work-summary ~/.kiro/crew/skills/
```

(If your install uses `$KIROCREW_HOME`, copy into `$KIROCREW_HOME/skills/` instead.)

Then in any chat, say "summarize my day" — the agent discovers the skill by its triggers.

### Optional: daily cron

Ask your agent to "run the daily-work-summary every day at 6pm as a dashboard
notification". It registers an agent cron that loads this skill each evening.

### Notes

- The skill classifies git repos as work vs personal by their **remote URL** — adjust to your setup.
- Jira/Confluence require a connected Atlassian account; a `401 scope does not match` is a stale token (re-authorize the connection), not a skill failure.
