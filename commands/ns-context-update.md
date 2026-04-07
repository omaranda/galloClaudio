---
name: ns-context-update
description: Update the shared AI context after an architectural decision, convention change, or domain knowledge addition — commit to the shared repo so the whole team gets the update.
---

You are updating the shared AI context for the Natural State team. This is used when an important decision or piece of knowledge should be available to all team members.

**Step 1 — Identify what changed:**
Ask the user (if not already clear):
- Is this an **architecture decision**? (goes to `memory/architecture.md`)
- Is this a **convention change**? (goes to `memory/team-conventions.md`)
- Is this a **domain knowledge update**? (goes to `memory/domain-knowledge.md`)
- Is this a **project status update**? (goes to `memory/active-projects.md`)
- Is this a **CLAUDE.md update**? (stack, repos, conventions sections)

**Step 2 — Read current state:**
- Read the target file to understand what's already there
- Avoid duplicating existing information

**Step 3 — Make the update:**
- For architecture decisions: follow the ADR format (number, date, status, description)
- For conventions: add to the appropriate section with clear rationale
- For domain knowledge: add under the relevant heading
- For project status: update the table row or add a new row
- For CLAUDE.md: update the relevant section

**Step 4 — Commit and push:**
- Stage only the changed files in `naturalstate-ai-context/`
- Commit with message: `context: [brief description of what changed]`
- Push to origin

**Step 5 — Notify:**
Tell the user: "Context updated. Team members should `git pull` naturalstate-ai-context to get the latest."
