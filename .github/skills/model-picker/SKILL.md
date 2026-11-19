---
name: model-picker
description: Choose which Claude model (Haiku 5.5, Sonnet 5.5, Opus 5.5, Fable 5.2) and effort level to use for a task, estimate cost and latency trade-offs, and decide how to mix models (subagents, advisor, orchestrator-workers). Use this whenever the user asks which Claude model to pick or build with, whether to move up or down a model tier, how to cut Claude API or Claude Code costs, what effort level to set, which model a subagent should run on, how to migrate between Claude models, or how to benchmark models on their own tasks. Also use it when designing any multi-step LLM pipeline, agent, or Claude Code subagent where a model has to be assigned per step, even if the user never says "model picker".
---

# Model picker

Based on Addy Osmani's "Which Claude model should you build with?" playbook (published 2026-11-09). Prices, defaults and benchmark figures are a snapshot of that article. Before giving a user a final cost number or committing a production config, check the current Claude docs (the `claude-api` skill covers model IDs and pricing), because list prices and defaults change.

## Core idea

Model choice is a trade-off between capability, cost per *finished task* (not per token), and latency. Effort level is a second, finer dial. Most of the value comes from starting in the right place and then moving one step at a time based on evidence, not from picking the "best" model up front.

## Default recommendation

Start on **Opus 5.5 at medium effort**. It is the Claude Code default and the platform-recommended starting point, at half the per-token price of Fable 5.2. Then move:

- **Down to Sonnet 5.5** for well-scoped everyday work (bug fix, feature against a spec, chat, everyday documents/slides/spreadsheets, repeated agent tasks). A clear spec alone is not enough: a migration or performance optimization can have a clear spec and still be hard, so keep it on Opus 5.5 (or Fable 5.2 if major).
- **Further down to Haiku 5.5** for the simplest high-volume work: classification, routing, tagging, extraction, and subagents doing cheap side tasks.
- **Up to Fable 5.2** for the hardest technical projects (coding, finance, sciences), major refactors/migrations, architecture planning, leading many subagents, deep research, long unattended runs, or when Opus 5.5 at high still falls short.

## Task lookup

| Task | Start with | Effort | Why |
|---|---|---|---|
| Everyday engineering, debugging, multi-PR features | Opus 5.5 | medium, then high | Balance of cost and capability |
| Classification, routing, tagging, high-volume extraction | Haiku 5.5 | medium | Lowest cost and latency |
| Subagents that search files, read logs, summarize | Haiku 5.5 or Sonnet 5.5 | model default | Wrong answers are cheap to spot; main model holds the judgment |
| Chat and interactive sessions | Sonnet 5.5 (Haiku 5.5 for simple chat at volume) | low or medium | Fast, especially at low/medium |
| Well-scoped coding (bug fixes, iteration, spec-driven) | Sonnet 5.5 | medium, high for harder | Half the Opus 5.5 token price |
| Everyday docs, slides, spreadsheets | Sonnet 5.5 | high (medium in Claude Code) | Iterates fast, good design eye. Complex financial models or big source piles: start Opus 5.5 |
| Repeated agent tasks (investigation, review, drafting) | Sonnet 5.5 | medium | Defined, repeated tasks rarely need more |
| Computer use, vision-heavy workflows | Opus 5.5 | medium | Long agentic loops over many screenshots |
| Major refactors/migrations, architecture, leading many subagents | Fable 5.2 | medium, then high | Most capable generally available model for technical fields |
| Deep research, scientific/quantitative analysis | Fable 5.2 | medium, then high | Built for the hardest technical work |
| Not sure yet | Opus 5.5 | medium | Safe default |

Claude Mythos 5.2 exists for cybersecurity and life-sciences research through verification programs only; it is not a general pick.

## Escalation order when a model underperforms

Follow these in order, because each step is cheaper than the next and effort cannot add knowledge the model lacks:

1. **Give it a way to verify its own work** (test, build, script that calls the endpoint). This usually costs less and adds less latency than anything else.
2. **Raise effort one level.** Helps when the model needs to think longer, call more tools, or self-check. Adds latency.
3. **Move up one model**, and start at a *lower* effort than you used on the smaller model. Past `high`, one more effort level can cost as much as a bigger model, so compare "next model at lower effort" against "same model at higher effort" on your own work.

Going the other way: one effort level down often gets the same result at a lower rate.

**Cheap-first, re-run failures:** if outcomes are automatically checkable, run everything at low effort and re-run only failures at high. On SWE-bench Pro, Opus 5.5 at low with failures re-run at high passed ~97% at ~$0.17/task vs 95.3% at $0.29 with every task at high.

## Trade-offs to reason about

- **Cost per finished task beats price per token.** Opus 5.5 at high beat the previous Fable at max on Terminal-Bench 3.0 (58.9% vs 58.0%) with half the tokens. On FrontierCode 2.0, Fable 5.2 at high matched Opus 5.5 at max (45.3% vs 45.5%) for roughly half the cost. So before pushing Opus 5.5 past high on a hard task, test Fable 5.2. Price the hardest tasks in a set too: on one benchmark, 2 of 20 problems carried 43% of spend.
- **Latency vs accuracy.** Decide how long a person can wait first. For replies a person waits on, start Sonnet 5.5 at low or medium. Opus 5.5 effort curve on SWE-bench Pro (vs high): medium ≈ -2.5 pts for ~70% of cost; low ≈ -8 pts for ~⅓ of cost; xhigh ≈ +1.4 pts for 2.5× cost. For speed without losing capability, Opus 5.5 and Fable 5.2 have fast mode (premium pricing).
- **Context vs cost.** All four models have 1M-token context, but each turn resends it; on long runs most input bills as cache reads. A lean main context matters more the bigger the main model is. Keep the expensive model's context for decisions and let cheap subagents do the verbose reading. Fable and Opus cache reads are both $0.20/M, so on cache-heavy runs the Fable/Opus gap comes down to uncached input and output (2× per token).
- **Haiku 5.5 pricing is tiered by prompt length** (see `references/models-and-pricing.md`). It pays off most on short prompts (a twentieth of Sonnet 5.5's input price). With consistently long prompts, compare cost per finished task against Sonnet 5.5, not list price. Small models do well on high-volume, checkable work and are weaker in long agentic loops.

## Mixing models

Read `references/mixing-models.md` when designing subagents, an advisor setup, or an orchestrator with workers. Key rules:

- **Switch at a natural break.** The prompt cache belongs to the model that wrote it, so the first turn on a new model pays cache-write on the whole conversation. In Claude Code run `/compact` first or start a fresh session with a short written plan. On the API, open with a summary instead of full history.
- **Moving down a tier loses reasoning** (a model never reads thinking blocks of a model above it). Moving from Fable 5.2 to any other model drops its reasoning. The request still succeeds.
- **Small models on cheap-to-check work** (find files, run tests, read logs); **code edits and judgment calls stay on the main model**, because a small model's misread sends the main model to the wrong file.
- **Simple routing rules belong in code**, not in a model asking itself which model to use.

## Compatibility checks before swapping models

Read `references/migration-checklist.md` before changing a model name in a request. The short version: thinking is always on for Opus 5.5 and Fable 5.2, `tool_choice` of `any`/`tool` returns 400 on Sonnet 5.5, Opus 5.5 and Fable 5.2, Fable 5.2 rejects requests with edited history before a thinking block, Haiku 4.5 → 5.5 needs `budget_tokens` replaced by an effort level, and computer use needs the `computer_toolset_20260801` toolset.

## Measure on the user's own work

Published numbers get a starting model; the user's tasks decide. Offer this procedure when the user is about to commit:

1. Sample real tasks and write an outcome check for each (passing test, closed ticket, correct row count).
2. Run each candidate model at two or three effort levels, each in its own session so runs don't share a cache. Record cost per completed task from each response's `usage` field.
3. Plot score against spend. Keep a multi-model setup only if it beats every single-model point.
4. Shadow-run the winner on a slice of real traffic before cutover.

Quick check in Claude Code: run one backlog task on two models via `/model` and compare `/usage`.

## How to answer the user

- Give a recommendation first (model + effort), then the one or two reasons that matter for *their* task, then what evidence would make them move up or down. Avoid surveying all four models when one answer fits.
- Ask about the deciding factors only when they are missing: how hard/ambiguous the task is, whether outputs can be checked automatically, volume, latency tolerance, and prompt/context length.
- Label figures as list prices from the article snapshot, and say when a number is a benchmark result on a specific benchmark rather than a guarantee for the user's workload.
- When the user is building something concrete (pipeline, subagents), output a concrete per-step assignment table: step, model, effort, reason, and the signal that would trigger escalation.
