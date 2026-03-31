# /ax-build

<!--
context: fork
argument-hint: "issue number"
-->

You are executing the `ax-build` skill. Your job is to implement one GitHub issue and open a pull request.

**Invocation:** `/ax-build [issue number]` — the issue number is required.

Follow these steps exactly, in order.

---

## Startup: Check for orphaned in-progress issues

Before touching the target issue, check for orphaned work:

1. List all issues on this repo labeled `in-progress` via GitHub MCP.
2. For each `in-progress` issue, check whether there is an open pull request referencing it.
3. Any `in-progress` issue with no open PR is orphaned — move it back to `build-lane` by replacing its `in-progress` label with `build-lane` via GitHub MCP.
4. Report what was recovered: "Recovered issue #N back to build-lane (no open PR found)."

---

## Step 1: Read the target issue

Read the full body of the target issue via GitHub MCP. If it is a sub-issue (its body references a parent epic issue number), also read the parent epic in full.

---

## Step 2: Move issue to in-progress

Replace the issue's current label with `in-progress` via GitHub MCP.

---

## Step 3: Read project context

Read `SPEC.md` and `CLAUDE.md` from the repo root. Focus on sections relevant to this issue's acceptance criteria and scope.

---

## Step 4: Create a feature branch

Create a branch named: `claude/epic-{issue-number}-{short-title}`

Where `{short-title}` is a 2–4 word kebab-case summary of the issue title.

Example: `claude/epic-42-add-user-auth`

---

## Step 5: Implement

Work on the feature branch. Commit frequently with clear, descriptive commit messages. Each commit should represent a logical unit of work.

Stay strictly within the scope defined by the issue's acceptance criteria. If you discover work that is needed but out of scope, note it — do not implement it here.

---

## Step 6: Rebase onto main

Before opening a PR, rebase the feature branch onto `main`:

```
git fetch origin
git rebase origin/main
```

**If rebase conflict:** Abort the rebase (`git rebase --abort`). Comment on the issue via GitHub MCP describing exactly what conflicted. Move the issue back to `build-lane`. Exit. Leave the repo in a clean state.

---

## Step 7: Self-evaluate

Review your implementation against each acceptance criterion in the issue. For each criterion, determine: Met / Partial / Not met. Produce an overall verdict:

- **Met** — all criteria fully satisfied
- **Partial** — most criteria met, with minor gaps (explain)
- **Not met** — one or more criteria not satisfied (explain why, and what would be needed)

---

## Step 8: Create follow-on issues

If you discovered any out-of-scope work during implementation that should be done later, create one GitHub issue per item via GitHub MCP, labeled `backlog`. Reference the current issue number in the body.

---

## Step 9: Open a pull request

Create a pull request via GitHub MCP:
- Base: `main`
- Head: the feature branch
- Title: concise description matching the issue title
- Body: use the template at `.claude/skills/ax-build/templates/pr-template.md`

---

## Step 10: Close the issue

Move the issue to `done` and close it via GitHub MCP.

If this was a sub-issue: check whether all sibling sub-issues (those referencing the same parent epic) are now labeled `done` or closed. If all siblings are done, close the parent epic issue and move it to `done` as well.

---

## Handling blocked work

If you cannot proceed because of a missing dependency or unresolvable blocker:

1. Comment on the issue via GitHub MCP explaining the blocker clearly.
2. If the blocker is actionable, create a follow-on issue for it labeled `backlog`.
3. Move the issue back to `backlog`.
4. Exit. Leave the repo in a clean state (no uncommitted changes, no partial branches pushed).

---

## Handling ambiguous acceptance criteria

If the acceptance criteria are ambiguous and cannot be resolved by reading SPEC.md or CLAUDE.md:

1. Comment on the issue via GitHub MCP requesting clarification, quoting the specific ambiguous criterion.
2. Move the issue back to `backlog`.
3. Exit.

---

## What you must not do

- Implement multiple issues in one invocation
- Merge your own PR
- Modify SPEC.md
- Make architectural decisions not documented in the issue or spec
- Leave the repo in a dirty state if you exit early
