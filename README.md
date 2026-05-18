# ai-rules

Personal Claude Code plugins for structured engineering work. Two plugins covering different scopes:

- **`flow`** — single-session implementation: plan → implement → review → ship
- **`harness`** — multi-session feature work: context handoff, plan worklogs, wrap-up

## Install

Clone this repo, then run in your terminal:

```
claude plugin marketplace add ~/workspace/personal/ai-rules
claude plugin install flow
claude plugin install harness
```

Then open a fresh Claude Code session for the plugins to be active.

---

## `flow` — single-session implementation

For focused, contained work that starts and finishes in one session.

### The flow

```
draft plan in plan mode
        │
        ▼
   /flow:plan-review     →  critiques plan, saves to .claude/plans/<name>.md
        │
        ▼
      /clear             (fresh context — recommended)
        │
        ▼
/flow:implement <plan>   →  pre-flight, branch from Linear ticket, step-by-step commits
        │
        ▼
   /flow:ship-loop       (auto-invoked by /flow:implement)
        │
        ├─ /flow:self-review
        ├─ /flow:pr               →  opens draft PR using repo template
        └─ /flow:review-loop      →  iterates until clean, then handles CI
```

For dependency vulnerability fixes, use the shortcut skill instead:

```
/flow:vuln-fix           →  fetches context from Linear, bumps package, raises PR
```

### Commands

| Command | Model | What it does |
| --- | --- | --- |
| `/flow:plan-review` | opus | Critiques and refines a plan, saves it to `.claude/plans/` |
| `/flow:implement <plan>` | sonnet | Pre-flight, branches from Linear ticket, executes plan with one commit per step |
| `/flow:ship-loop` | sonnet | Orchestrator: self-review → PR → review loop |
| `/flow:self-review` | sonnet | Strict pre-commit review of the current diff (lint, hygiene, correctness) |
| `/flow:pr` | sonnet | Opens a draft PR; uses repo PR template; pulls Linear + Figma context |
| `/flow:review-loop` | sonnet | Up to 5 iterations of fresh-context review and fix, then handles CI |
| `/flow:review-pr` | opus | Deep PR reviewer (security, correctness, maintainability) run in parallel |
| `/flow:vuln-fix` | sonnet | Dependency vulnerability fix: fetches from Linear, bumps package, raises PR |

### Loops

| Loop | Where | Cap | Purpose |
| --- | --- | --- | --- |
| Self-critique | inside `/flow:plan-review` | 3 iterations | tighten the plan before saving |
| Review / fix | inside `/flow:review-loop` | 5 iterations | drive blocking issues to zero |
| CI fix | inside `/flow:review-loop` | 2 attempts | bounded retry for green CI |

### Multiverse conventions

- `/flow:pr` uses the Multiverse PR template when the remote is `multiverse-io`; includes Linear card URL in the Links section
- `/flow:review-loop` never auto-marks Multiverse PRs as ready — leaves them as draft for manual review
- `/flow:implement` fetches the Linear `branchName` field to name the branch

---

## `harness` — multi-session feature work

For longer-running work that spans multiple sessions or Claude tabs.

### Commands

| Command | What it does |
| --- | --- |
| `/harness:warp` | Builds a copyable session summary to paste into a fresh `/clear`'d session |
| `/harness:wrap-up` | Closes out plan execution: marks steps done, captures lessons, updates CLAUDE.md |
| `/harness:plan-status` | Lists all active plans in `.claude/plans/` with progress and last activity |

### Worklog hook

The harness registers a `PostToolUse` hook that automatically organises plan files:

- New plans written to `.claude/plans/.tmp/<slug>.md` are moved into a dated folder: `.claude/plans/YYYY-MM-DD-<description>/plan.md`
- A `worklog.md` is auto-created alongside each plan
- Log progress during execution: `| YYYY-MM-DD HH:MM | action | detail |`
- If a step fails or the approach changes, log a `RE-PLAN` entry and edit `plan.md` in place — never create a new plan file

---

## Status line (optional)

`flow/scripts/statusline.sh` renders project + branch + model + context-usage bar in the Claude Code status line.

Wire it up manually in `~/.claude/settings.json`:

```bash
ln -s ~/workspace/personal/ai-rules/flow/scripts/statusline.sh ~/.claude/statusline-command.sh
```

```json
"statusLine": {
  "type": "command",
  "command": "bash ~/.claude/statusline-command.sh"
}
```

---

## Conventions

- Plans live in `.claude/plans/` at the repo root
- Branch names come from the Linear ticket's `branchName` field
- Fresh context (`claude --print`) is used for plan and PR reviews to avoid echo-chamber bias
- Conventional Commits for PR titles; ticket IDs go in the PR body Links section only, not the title
