# model-picker — choose the right Claude model

An Agent Skill that helps you pick the right Claude model (Haiku 5.5, Sonnet 5.5, Opus 5.5, Fable 5.2) and effort level for a task, estimate cost and latency, and decide how to mix models across subagents.

It works in **Claude Code** and **GitHub Copilot** (both read the Agent Skills `SKILL.md` format).

## The short version

| Task | Start with | Effort |
|---|---|---|
| Not sure yet | Opus 5.5 | medium |
| Classification, routing, extraction, cheap subagents | Haiku 5.5 | medium |
| Well-scoped coding, docs, slides, chat | Sonnet 5.5 | medium |
| Everyday engineering, debugging, computer use | Opus 5.5 | medium → high |
| Major refactors, architecture, deep research | Fable 5.2 | medium → high |

**When output falls short, escalate in this order:**
1. Give the model a way to verify its own work (test, build, script)
2. Raise effort one level
3. Move up one model, starting at a lower effort

Measure **cost per finished task**, not price per token.

## What's inside

```
.github/skills/model-picker/
├── SKILL.md                          # decision rules, task lookup, escalation order
└── references/
    ├── models-and-pricing.md         # prices, context, tiered Haiku pricing
    ├── mixing-models.md              # subagents, advisor, orchestrator-workers
    └── migration-checklist.md        # moving between model versions
```

## Install

**GitHub Copilot (one repo):** copy `.github/skills/model-picker/` into the root of your repo and commit. Copilot agent mode picks it up automatically.

**GitHub Copilot (all repos):** copy `model-picker/` to `~/.copilot/skills/`.

**Claude Code:** copy `model-picker/` to `~/.claude/skills/` (personal) or `.claude/skills/` (project).

Then just ask: *"Which Claude model should I use for this?"*

## Source and caveats

Based on Addy Osmani's "Which Claude model should you build with?" playbook (2026-11-09). Prices and benchmark numbers are a snapshot. Check the current Claude docs before committing a production config.

## License

MIT
