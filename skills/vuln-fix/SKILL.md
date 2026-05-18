---
name: vuln-fix
description: Fix a dependency vulnerability by bumping the affected package to the patched version. Use when the user runs `/flow:vuln-fix` or references a Linear vulnerability ticket (e.g. SYNC-xxxx) and asks to fix it.
user-invocable: true
---

Fix the dependency vulnerability referenced in the conversation. Do the following steps sequentially. After each step, report what you did.

## Step 1: Get context from Linear

Look for a Linear ticket reference in the conversation (e.g. `SYNC-1429`).
If found, fetch it with `mcp__claude_ai_Linear__get_issue` and extract:
- The package name and minimum patched version from the Fix section
- The Dependabot alert URLs from the Links section
- The suggested `branchName` field (use this for the branch)

If no ticket is found, ask the user for the package name, target version, and relevant CVE/GHSA links before continuing.

## Step 2: Check the current state

1. Run `git status` — working tree must be clean. If not, stop and ask.
2. Run `git branch --show-current` — if not on `main`, stop and confirm with the user.
3. Find the package in `package.json` and note the current version.
4. Confirm the package exists before proceeding.

## Step 3: Create a branch

Use the `branchName` from Linear if available.
Otherwise derive one from the ticket ID and package name (e.g. `sync-1429-fix-h3-vulnerabilities`).

```bash
git checkout -b <branch-name> origin/main
```

## Step 4: Bump the version

Update the version constraint in `package.json` to the minimum patched version (e.g. `^1.15.9`).

Then run:
```bash
yarn install
```

Verify the `yarn.lock` now resolves to a version that satisfies the patched requirement:
```bash
grep -A2 '"<package>@' yarn.lock | grep version
```

If Yarn 3/4 is in use (check `yarn --version`), note that `yarn upgrade` does not exist — edit `package.json` directly and run `yarn install`.

## Step 5: Check if build/tests pass on main first

Before running validation, verify the build isn't already broken on main:
```bash
git stash && yarn build 2>&1 | tail -5
git stash pop
```

If the build fails on main too, note it as a pre-existing issue and proceed — do not block the fix on an unrelated failure.

Run the relevant validation for the changed area if the build is healthy.

## Step 6: Commit

```bash
git add package.json yarn.lock
git commit -m "chore: upgrade <package> to <version> to fix <CVE/GHSA IDs>

<one sentence describing the vulnerability>

Resolves <LINEAR-TICKET-ID>.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

## Step 7: Push and raise PR

Push the branch and raise a PR following the repo's PR template (run `/flow:pr`).

For Multiverse repos the PR body should:
- Explain the vulnerability and what the upgrade fixes (no manual testing needed — state that explicitly)
- Check off checklist items that are N/A
- Include the Linear card URL and both Dependabot alert URLs in the Links section

## Important rules

- Never guess the patched version — get it from the Linear ticket or public advisory.
- Only bump the specific vulnerable package. Do not run a broad upgrade.
- If the package appears in `resolutions` or `overrides` as well as `dependencies`, update both.
- Do not block on pre-existing build failures unrelated to the fix.
