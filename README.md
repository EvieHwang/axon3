# Axon3

Axon3 is a portable Claude Code skills framework for spec-driven development. It provides three slash commands that translate operator intent into GitHub issues, implement those issues as pull requests, and induct new projects into the workflow.

## Loading Axon3

In any Claude Code session:

```bash
claude --add-dir ~/axon3
```

With SpecKit (if installed at `~/tools/spec-kit`):

```bash
claude --add-dir ~/axon3 --add-dir ~/tools/spec-kit
```

## Skills

### `/ax-prep`
Translates a spec change or operator intent into GitHub issues on the backlog.

Use when: you have edited `SPEC.md` or have a feature idea and want to create structured backlog issues before implementation begins.

### `/ax-build [issue number]`
Implements one issue from the build-lane and opens a pull request.

Use when: an issue has been moved to `build-lane` and is ready to implement.

### `/ax-init [owner/repo]`
Inducts a new repository into Axon3 governance.

Use when: setting up a new project. Installs the submodule, copies scaffolding files, creates labels, and sets up SPEC.md and CLAUDE.md.

## Inducting a new project

In a Claude Code session in the target project's directory:

```bash
claude --add-dir ~/axon3
```

Then run `/ax-init`. Follow the post-induction checklist it prints.

## Keeping skills current in inducted projects

When you update Axon3 and want managed projects to pick up the changes:

```bash
cd ~/your-project
git submodule update --remote .claude/skills/axon3
git commit -m "chore: update axon3 skills"
```
