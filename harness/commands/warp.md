---
description: Build a copy-pasteable summary of the current session to bootstrap a fresh context after /clear.
---

Collate a detailed overview of all work carried out this session: what was built, files changed, decisions made, and anything relevant for picking up where we left off. Then construct it into a prompt I can paste into a new session to fully restore context.

Steps:
1. Ensure any active plan is written to `.claude/plans/<plan-name>/plan.md` in the repo root.
2. If a `worklog.md` exists in that plan folder, ensure it is fully up to date.
3. Produce a single copyable block that includes:
   - A brief summary of what was accomplished
   - Key file paths touched
   - Current branch and any open PRs
   - Links to the plan and worklog
   - Any blockers or next steps
   - Enough context that a fresh session can continue without re-explaining anything
