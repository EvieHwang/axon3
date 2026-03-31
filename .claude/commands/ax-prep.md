# /ax-prep

You are executing the `ax-prep` skill. Your job is to translate a spec change or operator intent into one or more GitHub issues on the backlog.

Follow these steps exactly, in order.

---

## Step 1: Read context

Determine what has changed. Check in this order:

1. **Conversation context** — Has the operator just described an intent in the conversation immediately before this invocation? If yes, use that as the change description.
2. **SPEC.md diff** — Run `!git diff HEAD -- SPEC.md`. If it yields meaningful content, use that.
3. **Neither** — Ask the operator: "What change or intent should I create issues for?" Wait for their answer before proceeding.

---

## Step 2: Check the existing backlog

Read all open GitHub issues on this repo via the GitHub MCP. Identify any issues that already cover the described change. If the change is already fully represented in the backlog, tell the operator and stop — do not create duplicate issues.

---

## Step 3: Classify the change

Apply this decision rule:

**Use SpecKit path** when the change has ANY of the following:
- Architectural impact (changes how components interact)
- Introduces a new data model or system boundary
- Crosses multiple existing epics
- Would require more than one independent task to implement safely
- Has ambiguous design decisions that need resolution first

**Use Direct path** when ALL of the following are true:
- The change is self-contained
- It fits one feature branch
- No ambiguous design decisions

**Examples requiring SpecKit:** adding a new API layer, changing the authentication model, introducing a new database table, refactoring a core abstraction.

**Examples handled directly:** adding a new endpoint to an existing API, updating validation logic, adding a UI component within an established pattern.

---

## Step 4: Present classification and ask for confirmation

Tell the operator:
- Which path you chose (Direct or SpecKit)
- A one-line rationale

Example: *"I'm classifying this as a **direct path** change — it adds one new endpoint to an existing API with no design ambiguity."*

Ask: "Confirm this classification, or override to the other path?"

Wait for their response before proceeding.

---

## Step 5: Execute the chosen path

### Direct path

1. Create one parent epic issue on this repo via GitHub MCP, labeled `backlog`.
   - Title: a clear, concise description of the epic
   - Body: use the template at `.claude/skills/ax-prep/templates/epic-template.md`

2. Assess whether the epic naturally decomposes into discrete, independently committable tasks. Create sub-issues **only** when there is a genuine reason each task could be completed and reviewed separately. Prefer fewer, larger sub-issues over many small ones.

   If sub-issues are warranted:
   - Create each sub-issue via GitHub MCP, labeled `backlog`
   - Body: use the template at `.claude/skills/ax-prep/templates/task-template.md`
   - Include the parent epic issue number in each sub-issue body

3. Stop. Tell the operator what was created and that they can move issues to `build-lane` when ready to implement.

### SpecKit path

Tell the operator: "This change needs SpecKit. Run `/speckit.specify` next (load SpecKit with `--add-dir ~/tools/spec-kit`). Once SpecKit produces a refined spec, return here and I will create the parent epic and sub-issues."

Do **not** invoke SpecKit yourself. Stop.

---

## What you must not do

- Make the SpecKit/Direct decision without operator confirmation
- Create issues for changes already covered by open backlog issues
- Prioritize or order the backlog
- Proceed past Step 4 without operator confirmation
