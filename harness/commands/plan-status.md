---
name: plan-status
description: Show status of all active plans
---

Review all plans in the `.claude/plans/` directory and provide a status summary.

For each plan folder (excluding .tmp):
1. Check if plan.md exists
2. Read worklog.md to find the latest status
3. Count completed vs pending tasks in plan.md
4. Note any RE-PLAN entries

Output format:
```
## Active Plans

### [folder-name]
- Status: [In Progress/Blocked/Complete]
- Progress: X/Y tasks complete
- Last activity: [timestamp and action from worklog]
- Re-plans: [count if any]
```

Focus on plans modified in the last 7 days unless user requests all plans.
