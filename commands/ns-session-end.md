---
name: ns-session-end
description: End a Natural State work session — generate daily report, update plans and Jira tasks, commit changes to the shared AI context repo.
---

You are closing a work session for the Natural State team. Follow these steps:

**Step 1 — Gather session activity:**
- Review the conversation history to identify what was accomplished
- Check `git status` and `git log` across active repos to see what changed
- List any decisions made, blockers encountered, or next steps identified

**Step 2 — Update Jira tasks (if MCP connected):**
- For each task worked on today, update its status in Jira (In Progress, Done, etc.)
- Add comments to tasks with relevant details or decisions
- If Jira MCP is not connected, skip and note which tasks need manual update

**Step 3 — Update plans:**
- For any plan in `naturalstate-ai-context/plans/`, check off completed steps
- Update `naturalstate-ai-context/memory/active-projects.md` with current status

**Step 4 — Generate daily report:**
- Create `naturalstate-ai-context/daily/YYYY/MM/DD.md` with today's date
- Use this format:

```markdown
# Daily Report — YYYY-MM-DD

## Accomplished
- [bullet list of what was done]

## Decisions
- [any architectural or product decisions made]

## Blockers
- [anything blocking progress]

## Next Steps
- [what should be done in the next session]

## Repos Touched
- [list of repos with changes]
```

**Step 5 — Commit and push:**
- Stage all changes in `naturalstate-ai-context/`
- Commit with message: `daily: YYYY-MM-DD session report`
- Push to origin

**Step 6 — Present closing summary:**
Keep it to 10 lines max. The report file has the details.
