# /ax-init

You are executing the `ax-init` skill. Your job is to induct a new project into Axon3 governance.

**Invocation:** `/ax-init [owner/repo]` — the repo argument is optional. If omitted, use the current repo.

Follow these steps exactly, in order.

---

## Step 1: Determine the target repo

If an `owner/repo` argument was provided, use that. Otherwise, determine the current repo from `git remote get-url origin`.

All GitHub operations in the following steps target this repo.

---

## Step 2: Add Axon3 as a Git submodule

In the target project, add the Axon3 repo as a submodule:

```
git submodule add https://github.com/EvieHwang/axon3 .claude/skills/axon3
```

This makes all three Axon3 skills (`/ax-prep`, `/ax-build`, `/ax-init`) available in any Claude Code session run from the target project via `--add-dir .claude/skills/axon3`.

---

## Step 3: Copy scaffolding files

Copy the contents of the `scaffolding/` directory from the Axon3 repo into the target project root. The scaffolding directory contains:

- `.github/workflows/deploy.yml`
- `.gitignore`
- `.axon/.gitkeep`

For each file: if the file already exists in the target project, skip it and add it to a "skipped files" list. Do not overwrite existing files.

After copying, report any skipped files to the operator.

---

## Step 4: Create SPEC.md

If `SPEC.md` does not already exist in the target project root, create it using the template at `.claude/skills/ax-init/templates/spec-template.md`.

If `SPEC.md` already exists, skip this step.

---

## Step 5: Create CLAUDE.md

If `CLAUDE.md` does not already exist in the target project root, create it using the template at `.claude/skills/ax-init/templates/claude-template.md`.

If `CLAUDE.md` already exists, skip this step.

---

## Step 6: Create Kanban labels

Create the four Axon3 Kanban labels on the target repo via GitHub MCP:

| Label | Color | Description |
|-------|-------|-------------|
| `backlog` | `#CCCCCC` | Created, not yet scheduled |
| `build-lane` | `#0075CA` | Approved for implementation |
| `in-progress` | `#FBCA04` | Being worked on now |
| `done` | `#0E8A16` | Merged and closed |

If a label already exists with the correct name, skip it.

---

## Step 7: Update .gitignore

Check whether `.axon/` is already listed in the target project's `.gitignore`. If not, append `.axon/` to it. If no `.gitignore` exists, create one containing `.axon/`.

---

## Step 8: Commit

Create a commit in the target project with all the changes from steps 2–7:

```
chore: induct project into axon3 governance
```

---

## Step 9: Print the post-induction checklist

Tell the operator:

---

**Induction complete. Next steps:**

1. **Fill in SPEC.md** — Open `SPEC.md` and fill in:
   - The `Constraints` zone: non-negotiable rules, stack choices, explicit prohibitions
   - The `Operator zone`: vision, features, acceptance criteria

2. **Fill in CLAUDE.md** — Open `CLAUDE.md` and fill in:
   - Stack (tech stack, one line each)
   - Key commands (test, lint, build)
   - Constraints (paste from SPEC.md Constraints zone)

3. **Add GitHub Actions secrets** — In your repo settings, add these secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_REGION`
   Then open `.github/workflows/deploy.yml` and replace the placeholder deploy command with your actual deploy command.

4. **Run `/ax-prep`** when ready to create your first epics.

---

## What you must not do

- Create any GitHub issues
- Make architectural decisions about the project
- Overwrite files that already exist
- Proceed if the submodule add fails — report the error and stop
