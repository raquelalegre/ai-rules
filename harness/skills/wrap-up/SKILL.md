---
name: wrap-up
description: "Close out plan execution. Finalises the plan, captures lessons, and produces a clean artifact. Use when the user says 'wrap up', 'we're done', 'finish up', 'close this out', or after completing all plan steps."
allowed-tools: "Read, Edit, Write, Glob, Grep, Bash(git diff *), Bash(git log *), Bash(ls *), AskUserQuestion"
user-invocable: true
---

Close out the current plan execution. You are running in the session that did the work, so you have the full conversation history available.

Announce at start: "Wrapping up — reviewing plan and session."

## Step 1: Find the Active Plan

Find the most recently modified plan folder:

```bash
ls -t .claude/plans/*/plan.md 2>/dev/null | head -5
```

Cross-reference with conversation history — you know which plan you've been working on. If ambiguous, ask the user.

Read both `plan.md` and `worklog.md` from the plan folder.

## Step 2: Finalise the Plan

Review what actually happened during this session (conversation history, git diff, worklog entries) against what the plan said.

In `plan.md`:
- Mark completed steps as **DONE**
- Mark abandoned steps with ~~strikethrough~~ and a brief reason
- If a `## Revision Log` section exists, confirm it's current
- If not, add one summarising any mid-session re-plans (check worklog for `RE-PLAN` entries and review conversation for moments where the approach changed)
- Add a status line at the top, right after the title: `**Status: COMPLETE — YYYY-MM-DD**` (or `PARTIAL` if not everything finished)

In `worklog.md`:
- Append a final row: `| YYYY-MM-DD HH:MM | COMPLETE | plan wrapped up via /harness:wrap-up |`

## Step 3: Capture Lessons

Review the session for patterns worth preserving. Sources to check:

1. **Conversation history** — user corrections, pushback, surprises, validated approaches
2. **Worklog RE-PLAN entries** — what went wrong and what we learned
3. **Git diff** — what actually changed vs. what was planned
4. **Things that worked well** — approaches worth repeating

For each lesson, draft a one-line addition for CLAUDE.md. Apply the bar:
- **Include**: recurring patterns, mistakes to avoid, discovered conventions, validated approaches
- **Exclude**: one-off fixes, project-specific trivia, things already documented
- **Route**: if a lesson fits a topic-specific standards file the project maintains, route it there instead of the root CLAUDE.md

Present proposed CLAUDE.md additions to the user with a brief "why" for each. Apply only what they approve.

## Step 4: Summary

Print a wrap-up:

```
## Session Complete

**Plan**: [plan folder name]
**Status**: COMPLETE | PARTIAL (N of M steps done)
**Lessons captured**: [count added to CLAUDE.md, or "none"]
**Revisions during execution**: [count of RE-PLAN entries, or "none"]
```

## Rules

- Don't fabricate lessons. Only capture things that actually happened in this session.
- Keep the plan readable — a new reader should understand the final state without knowing the history.
- The worklog and revision log carry the history. The plan body carries the final state.
- If nothing went wrong and there are no lessons, say so. Don't force insights.
