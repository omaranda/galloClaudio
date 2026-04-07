---
name: ns-session-start
description: Start a Natural State work session — pull latest shared AI context, check active Jira tasks, and set up the working context for today's session.
---

You are starting a new work session for the Natural State team. Follow these steps:

**Step 1 — Pull latest shared context:**
- Run `git pull` in `naturalstate-ai-context/` to get the latest team context
- Read `naturalstate-ai-context/CLAUDE.md` to load cross-project context
- Read `naturalstate-ai-context/memory/active-projects.md` to understand current work

**Step 2 — Check Jira (if MCP connected):**
- Use the Atlassian MCP tools to list open tasks assigned to the current user
- If Jira MCP is not connected, skip this step and note it

**Step 3 — Check recent activity:**
- Run `git log --oneline -10` across the repos in the workspace to see recent commits
- Read the most recent daily report in `naturalstate-ai-context/daily/` if it exists

**Step 4 — Present a session briefing:**
Format the output as:

```
## Session Briefing — [today's date]

### Active Projects
[from active-projects.md]

### Open Jira Tasks (if available)
[from Jira query]

### Recent Activity
[from git logs]

### Suggested Focus
[based on the above, suggest what to work on today]
```

Keep the briefing concise — no more than 30 lines. The goal is to orient the user quickly so they can decide what to work on.
