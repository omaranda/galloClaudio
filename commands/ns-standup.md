---
name: ns-standup
description: Generate a team standup summary — read active Jira tasks, recent daily reports, and plan status to produce a concise team update.
---

You are generating a standup summary for the Natural State team.

**Step 1 — Read current state:**
- Read `naturalstate-ai-context/memory/active-projects.md`
- Read the last 3 daily reports in `naturalstate-ai-context/daily/` (most recent first)
- Read any plans in `naturalstate-ai-context/plans/` that are `in-progress`

**Step 2 — Query Jira (if MCP connected):**
- List tasks with status "In Progress" across all NS/NIP projects
- List tasks completed in the last 7 days
- List any blocked or overdue tasks
- If Jira MCP is not connected, use the daily reports and plan files instead

**Step 3 — Generate standup:**
Format as:

```markdown
## Team Standup — [today's date]

### What's In Progress
| Initiative | Owner | Current Task | Status |
|---|---|---|---|

### Completed (last 7 days)
- [bullet list]

### Blockers
- [bullet list, or "None"]

### This Week's Focus
- [2-3 key items the team should prioritize]
```

Keep it under 30 lines. This is for quick team alignment, not a detailed report.
