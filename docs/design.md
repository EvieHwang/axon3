# Axon4 — Build Specification

You are building Axon4: a portable Claude Code skills framework for spec-driven development. Read this document completely before doing anything. Then execute the phases in order, committing after each epic. Ask before making any decision not covered here.

---

## What you are building

Three slash commands that can be loaded into any Claude Code session via `--add-dir`, and embedded into managed projects as a Git submodule:

- `/ax-prep` — translates a spec change or operator intent into one or more GitHub issues on the backlog
- `/ax-build` — implements one issue from the build-lane and opens a PR
- `/ax-init` — inducts a new repo into Axon governance

No runtime. No orchestrator. No code. Skills are Markdown only.

All GitHub operations use the GitHub MCP connector. Never use the `gh` CLI for primary operations.

---

## Repo structure you are creating

```
axon4/
├── README.md
├── CLAUDE.md
├── .gitignore
├── .claude/
│   └── skills/
│       ├── ax-prep/
│       │   ├── SKILL.md
│       │   └── templates/
│       │       ├── epic-template.md
│       │       └── task-template.md
│       ├── ax-build/
│       │   ├── SKILL.md
│       │   └── templates/
│       │       └── pr-template.md
│       └── ax-init/
│           ├── SKILL.md
│           └── templates/
│               ├── spec-template.md
│               └── claude-template.md
├── scaffolding/
│   ├── .github/
│   │   └── workflows/
│   │       └── deploy.yml
│   ├── .gitignore
│   └── .axon/
│       └── .gitkeep
└── docs/
    └── build-spec.md               # this file
```

---

## Kanban labels

Every project managed by Axon4 uses exactly four labels:

| Label         | Color           | Meaning                     |
|---------------|-----------------|-----------------------------|
| `backlog`     | grey `#CCCCCC`  | Created, not yet scheduled  |
| `build-lane`  | blue `#0075CA`  | Approved for implementation |
| `in-progress` | yellow `#FBCA04`| Being worked on now         |
| `done`        | green `#0E8A16` | Merged and closed           |

`ax-init` creates these on the target repo. They must also exist on the axon4 repo itself before you begin building — create them now via GitHub MCP before writing any skill files.

---

## The workflow these skills implement

```
Operator updates SPEC.md or describes intent in conversation
        │
        ▼
  /ax-prep
        │
   ┌────┴──────────────────────────┐
   ▼                               ▼
Direct path                   SpecKit path
(self-contained change,        (architectural impact,
 one feature branch)           new system boundary,
        │                      multi-task scope)
        │                           │
  One epic issue              Invokes /speckit.specify
  created on backlog          then creates parent epic
                              with sub-issues on backlog
        └────────────────────────────┘
                     │
              Process stops.
              Operator reviews backlog,
              moves chosen issue to build-lane.
                     │
                     ▼
           /ax-build [issue number]
                     │
        feature branch → implementation → PR
                     │
        Operator reviews and approves PR
                     │
        GitHub Actions: deploy and merge
                     │
                   repeat
```

---

## Epic 1: `/ax-prep`

**File:** `.claude/skills/ax-prep/SKILL.md`

### What this skill does

1. Reads context from one of two sources, in this order of preference:
   - The conversation immediately preceding invocation (if the operator has just described an intent)
   - `!git diff HEAD -- SPEC.md` (if the operator has just edited the spec)
   - If neither yields meaningful content, asks the operator what changed before proceeding
2. Reads all open GitHub issues via MCP to understand the current backlog
3. Classifies the change using this decision rule:

   > **Use SpecKit** when the change has architectural impact, introduces a new data model or system boundary, crosses multiple existing epics, or would require more than one independent task to implement safely.
   >
   > **Handle directly** when the change is self-contained, fits one feature branch, and has no ambiguous design decisions.
   >
   > Examples requiring SpecKit: adding a new API layer, changing the authentication model, introducing a new database table, refactoring a core abstraction.
   >
   > Examples handled directly: adding a new endpoint to an existing API, updating validation logic, adding a UI component within an established pattern.

4. Presents the classification and a one-line rationale to the operator
5. Asks the operator to confirm or override before proceeding
6. Executes the chosen path:
   - **Direct path:** creates one parent epic issue labeled `backlog` using `templates/epic-template.md`. If the epic naturally decomposes into discrete tasks (each committable independently), creates sub-issues using `templates/task-template.md` and links them to the parent. Then stops.
   - **SpecKit path:** tells the operator to run `/speckit.specify` next (SpecKit is a separate skill loaded via `--add-dir ~/tools/spec-kit`). Does not invoke it directly.

### Sub-issue guidance

Create sub-issues when the work has multiple independently committable steps. Each sub-issue should be the size of one logical commit. Prefer fewer, larger sub-issues over many small ones — only decompose when there is a genuine reason a task could be completed and reviewed separately.

When sub-issues are created, the parent epic is complete when all sub-issues are labeled `done`.

### Epic template

**File:** `.claude/skills/ax-prep/templates/epic-template.md`

```markdown
## Problem
[What requirement or behavior this addresses]

## Acceptance criteria
- Given [context], when [action], then [outcome]
- Given [context], when [action], then [outcome]

## Spec reference
[Section of SPEC.md this traces to, or "operator intent" if from conversation]

## Scope
[Explicit statement of what is and is not included]

## Dependencies
- Blocked by: #[issue] [if applicable]
- Blocks: #[issue] [if applicable]
```

### Task template

**File:** `.claude/skills/ax-prep/templates/task-template.md`

```markdown
## Parent epic
#[parent issue number]

## What to implement
[One paragraph. Precise enough to implement without re-reading the parent.]

## Acceptance criteria
- Given [context], when [action], then [outcome]

## Done when
[Specific, observable condition — e.g., "test passes", "endpoint returns 200", "component renders correctly"]
```

### What this skill does not do

- Make the SpecKit/direct decision unilaterally. It proposes; the operator confirms.
- Create issues for changes that are already covered by open backlog issues. Check first.
- Prioritize or order the backlog.

---

## Epic 2: `/ax-build`

**File:** `.claude/skills/ax-build/SKILL.md`

**Argument:** issue number (required). Invoked as `/ax-build 42`.

**Skill metadata:**
```
context: fork
argument-hint: "issue number"
```

### What this skill does

**On startup:**

1. Checks for orphaned `in-progress` issues — issues labeled `in-progress` with no open PR. Moves any found back to `build-lane`. Reports what was recovered.
2. Reads the target issue body in full, including its parent epic if it is a sub-issue.

**Implementation:**

1. Moves the issue to `in-progress` via MCP
2. Reads SPEC.md and CLAUDE.md for project context, focusing on sections relevant to the issue
3. Creates a feature branch: `claude/epic-{number}-{short-title}`
4. Implements the work, committing frequently with clear messages
5. Rebases the feature branch onto `main` before opening a PR
6. Self-evaluates against acceptance criteria: **Met / Partial / Not met**
7. Creates follow-on issues (labeled `backlog`) for any out-of-scope work discovered during implementation
8. Opens a PR using `templates/pr-template.md`
9. Moves the issue to `done` and closes it
10. If the closed issue is a sub-issue, checks whether all sibling sub-issues are now `done`. If so, closes the parent epic and moves it to `done`.

**If blocked:** Comments on the issue explaining the blocker. Creates a follow-on issue for the blocker if appropriate. Moves issue back to `backlog`. Exits. The repo is always left in a clean state.

**If rebase conflict:** Aborts the rebase. Comments on the issue describing what conflicted. Moves the issue back to `build-lane`. Exits.

**If acceptance criteria are ambiguous:** Comments on the issue requesting clarification. Moves it back to `backlog`. Exits.

### PR template

**File:** `.claude/skills/ax-build/templates/pr-template.md`

```markdown
## Closes
#[issue number]

## What was built
[One paragraph]

## What was not addressed
[Explicit scope boundaries — or "nothing, fully addressed"]

## Follow-on issues
[List of new issues created, or "none"]

## Self-evaluation
**[Met / Partial / Not met]** — [one sentence of reasoning]
```

### What this skill does not do

- Implement multiple issues in one invocation
- Merge its own PR
- Modify SPEC.md
- Make undocumented architectural decisions

---

## Epic 3: `/ax-init`

**File:** `.claude/skills/ax-init/SKILL.md`

**Argument:** optional GitHub repo name in `owner/repo` format. If omitted, uses the current repo.

### What this skill does

1. Adds the Axon4 repo as a Git submodule at `.claude/skills/axon4/` in the target project:
   ```
   git submodule add https://github.com/EvieHwang/axon4 .claude/skills/axon4
   ```
   This makes all three Axon4 skills available in any Claude Code session run from the target project.

2. Copies the contents of `scaffolding/` from the Axon4 repo into the target project root. Does not overwrite files that already exist — reports any skipped files.

3. Creates `SPEC.md` from `templates/spec-template.md` if it does not already exist.

4. Creates `CLAUDE.md` from `templates/claude-template.md` if it does not already exist.

5. Creates the four Kanban labels on the target repo via GitHub MCP.

6. Adds `.axon/` to `.gitignore` if not already present.

7. Makes an initial commit: `chore: induct project into axon4 governance`.

8. Tells the operator what to do next:
   - Fill in the Constraints and Operator zones in SPEC.md
   - Fill in stack and key commands in CLAUDE.md
   - Add AWS credentials to GitHub Actions secrets (lists the required secret names from `deploy.yml`)
   - Run `/ax-prep` when ready to create the first epics

### SPEC.md template

**File:** `.claude/skills/ax-init/templates/spec-template.md`

```markdown
# [Project name]

**Version:** 1.0
**Last updated:** [date]

---

## Constraints
[Non-negotiable rules: stack choices, architectural limits, explicit prohibitions.
Agents treat this zone as read-only with higher authority than the rest of the spec.
Example: "All state lives in the database. No in-memory state between requests."
Example: "No new npm dependencies without operator approval."]

---

## Operator zone
[Vision, features, acceptance criteria.
Written and maintained by the operator.
Agents read this and execute against it. Agents never modify this zone.]

---

## System state
[Feature statuses: planned / in progress / complete.
Architecture descriptions. Current repo structure.
Updated by agents to reflect reality:
- ax-build marks features in-progress when picked up
- ax-build marks features complete when PR merges]
```

### CLAUDE.md template

**File:** `.claude/skills/ax-init/templates/claude-template.md`

```markdown
# [Project name] — Claude context

## Stack
[Tech stack, one line each]

## Key commands
- Test: [command]
- Lint: [command]
- Build: [command]
- Deploy: handled by GitHub Actions on PR merge

## Kanban labels
- `backlog` — not yet scheduled
- `build-lane` — approved for implementation
- `in-progress` — currently being built
- `done` — merged and closed

## Branch naming
`claude/epic-{number}-{short-title}`

## Axon4 skills
Loaded via submodule at `.claude/skills/axon4/`. Update with:
`git submodule update --remote .claude/skills/axon4`

## Constraints
[Paste from SPEC.md Constraints zone]
```

Target: under 60 lines. If it grows beyond that, move content to SPEC.md.

### What this skill does not do

- Create any GitHub issues
- Make architectural decisions about the project
- Overwrite existing files

---

## Epic 4: `scaffolding/` directory

**File:** `scaffolding/.github/workflows/deploy.yml`

Write a GitHub Actions workflow that:

1. Triggers on PR merge to `main`
2. Checks out the repo
3. Configures AWS credentials from secrets (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`)
4. Runs a deploy step using a placeholder command (`# TODO: replace with your deploy command`) that `ax-init` instructs the operator to fill in
5. Has clearly labelled sections and inline comments explaining each step

The workflow must use standard, current GitHub Actions syntax. Use `actions/checkout@v4` and `aws-actions/configure-aws-credentials@v4`.

Also create:
- `scaffolding/.gitignore` — sensible defaults for Node/Python/macOS/AWS
- `scaffolding/.axon/.gitkeep` — ensures the `.axon/` runtime directory exists after induction

---

## Epic 5: README and CLAUDE.md for the axon4 repo itself

**README.md** — write a concise document covering:

- What Axon4 is (two sentences)
- How to load into any session: `claude --add-dir ~/axon4`
- How to load with SpecKit: `claude --add-dir ~/axon4 --add-dir ~/tools/spec-kit`
- The three skills and when to use each
- How to induct a new project (`/ax-init`)
- How skills stay current in inducted projects (`git submodule update --remote .claude/skills/axon4`)

**CLAUDE.md** — for sessions developing Axon4 using Axon4:

- Stack: Markdown, Claude Code skills, GitHub Actions YAML
- Key commands: none (no build step)
- Kanban labels as above
- Branch naming as above
- Note: all three skills are available in this session via the repo's own `.claude/skills/`

---

## Constraints for this build

- All GitHub operations via MCP. No `gh` CLI for primary operations.
- No Python, no shell scripts, no runtime code. Skills and templates are Markdown only. The exception is `scaffolding/deploy.yml` which is YAML.
- No skill references files outside its own directory. The one exception: `ax-init` reads from `scaffolding/` and `templates/` — both are within the axon4 repo, which is the session's root.
- Commit after each epic with a clear message.
- If something in this document is ambiguous or contradictory, stop and ask.

---

## Operator setup instructions

Complete these steps before running a build session.

### 1. Create the axon4 repo on GitHub

Create a new public repo: `EvieHwang/axon4`. Do not initialise with any files — the build session will create everything.

### 2. Clone locally

```bash
git clone https://github.com/EvieHwang/axon4.git ~/axon4
cd ~/axon4
```

### 3. Start the build session

```bash
claude --add-dir ~/axon4
```

Paste this build spec document and say: *"Build Axon4 per this spec. Start with Epic 0: create the four Kanban labels on this repo via GitHub MCP, then proceed through the epics in order."*

### 4. After the build session

Once all epics are committed and pushed:

- Open `scaffolding/.github/workflows/deploy.yml` and fill in your actual deploy command
- That's it — the rest is filled in by `ax-init` when you induct each project

### 5. Inducting a new project

In a Claude Code session in the target project's directory:

```bash
claude --add-dir ~/axon4
```

Then run `/ax-init`. Follow the post-induction checklist it prints.

### 6. Keeping skills current in inducted projects

When you update Axon4 skills and want managed projects to pick up the changes:

```bash
cd ~/your-project
git submodule update --remote .claude/skills/axon4
git commit -m "chore: update axon4 skills"
```

### 7. Loading SpecKit alongside Axon4

If SpecKit is installed at `~/tools/spec-kit`:

```bash
claude --add-dir ~/axon4 --add-dir ~/tools/spec-kit
```

`/ax-prep` will hand off to `/speckit.specify` when the SpecKit path is chosen.
