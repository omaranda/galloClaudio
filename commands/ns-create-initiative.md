---
name: ns-create-initiative
description: Create a new initiative — brainstorm, write a plan, create Jira Epic and tasks, and save the plan to the shared AI context repo.
---

You are creating a new initiative for the Natural State team. This is a structured process that goes from idea to actionable Jira tasks.

**Step 1 — Understand the initiative:**
- Ask the user to describe the initiative in 1-2 sentences if not already provided
- Ask: Who is the lead? What is the deadline (if any)? Which repos are involved?

**Step 2 — Brainstorm:**
- Explore the problem space: what are the requirements, constraints, and risks?
- Consider the existing architecture (read `naturalstate-ai-context/memory/architecture.md`)
- Identify dependencies on other active projects (read `naturalstate-ai-context/memory/active-projects.md`)
- Present 2-3 possible approaches with trade-offs
- Let the user choose the approach

**Step 3 — Write the plan:**
- Read `naturalstate-ai-context/plans/template.md` for the format
- Create a new file: `naturalstate-ai-context/plans/PENDING/plan-[short-name].md`
- Fill in all sections: Context, Approach, Steps, Dependencies, Verification, Risks
- Steps should be concrete and actionable (each one becomes a Jira task)

**Step 4 — Create Jira Epic and Tasks (if MCP connected):**
- Create a Jira Epic with the initiative name
- Create one Jira Task per step in the plan, linked to the Epic
- Update the plan file: replace `PENDING` Jira Epic key with the actual key
- Move the plan file from `plans/PENDING/` to `plans/{EPIC-KEY}/plan.md`
- If Jira MCP is not connected, skip this step and leave the plan in `plans/PENDING/`

**Step 5 — Update shared context:**
- Update `naturalstate-ai-context/memory/active-projects.md` with the new initiative
- Commit and push all changes to `naturalstate-ai-context`

**Step 6 — Present summary:**
```
## New Initiative Created

**Name:** [initiative name]
**Jira Epic:** [key or PENDING]
**Plan:** naturalstate-ai-context/plans/{key}/plan.md
**Tasks:** [count] tasks created
**Next step:** [first task from the plan]
```
