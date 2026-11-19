# Mixing models

## Switching mid-work

- The prompt cache belongs to the model that wrote it. The first turn on a new model pays the cache-write price on the whole conversation.
- Shrink that turn: run `/compact` in Claude Code first, or start a fresh session with a short written plan. On the API, begin the new model's conversation with a summary of the work so far.
- Thinking blocks record which model produced them, and each model reads only some others' blocks (Claude API and Claude Models on Google Cloud, Google managed):
  - Opus 5.5 reads Sonnet 5.5 and Haiku 5.5 blocks.
  - Sonnet 5.5 reads Haiku 5.5 blocks.
  - Fable 5.2 reads Opus 5.5, Sonnet 5.5 and Haiku 5.5 blocks.
  - On the Claude API, Fable 5.1 reads Opus 5.5 blocks.
- If the new model cannot read the old blocks, the API drops them and the request still succeeds; the model just works without that reasoning. Any move down a tier loses reasoning, and a move from Fable 5.2 to any other generally available model drops its reasoning.

## Subagents on smaller models

A subagent has its own context window and returns a summary; you pay at the rate of the model it runs on.

In Claude Code, set `model: haiku` or `model: sonnet` in the subagent definition. `CLAUDE_CODE_SUBAGENT_MODEL` sets the default for subagents that don't specify one.

Example `.claude/agents/log-reader.md`:

```markdown
---
name: log-reader
description: Reads test output and logs, and reports each failing case. Use after a test run.
tools: Read, Grep, Glob, Bash
model: haiku
---

Read the logs you're given. For each failure, report the file, the test name and the first error line. Don't edit any files.
```

Guidance:
- Small models: work where a mistake is cheap to spot (finding files, running tests, reading logs).
- Main model: code edits and judgment calls.
- The built-in Explore subagent runs on the session's model in current Claude Code versions (capped at Opus on the Claude API). To keep exploration cheap, define your own subagent named `Explore` with `model: haiku`.
- The `haiku` alias resolves to Haiku 5.5 on the Anthropic API (needs Claude Code v2.1.293 or later). On Amazon Bedrock, Google Cloud, Microsoft Foundry and Claude Platform on AWS it still resolves to Haiku 4.5. Pin it with `ANTHROPIC_DEFAULT_HAIKU_MODEL`.
- `opusplan` alias: plans with Opus in plan mode, executes with Sonnet. This puts code edits on Sonnet, the opposite of the advice above, so measure it on your own tasks before making it a default.

## Advisor tool (beta)

A faster, cheaper executor model runs the task and can ask a stronger advisor model for guidance mid-task. Most tokens bill at the executor's rate.

- Helps only when the advisor adds capability the executor lacks AND the executor actually asks (it asks less at low effort). Measure the executor's consult rate first.
- Best candidate: agentic work where most turns are routine.
- The platform cost guide is cautious: of four pairings in its table, none clearly beat the executor alone.
- Check accepted pairings. Sonnet 5.5 executor accepts Fable 5.2, Opus 5.5, Opus 5, Sonnet 5.5 as advisor. A Fable 5.2 executor accepts only Fable 5.2 or Mythos 5.2.
- Example of low payoff: Opus 5.5 high + Fable 5.1 advisor, 90.1% at $2.92 per attempt, +1.7 pts for ~2.1× cost, comparable to what raising effort buys.

## Orchestrator with cheap workers

For work bigger than one context window: a top-tier model leads, splits the task, hands pieces to many cheaper workers in parallel, merges results. Example: Opus 5.5 lead over up to 25 Sonnet 5.5 workers on a ~22M-token defect search cost ~⅓ less than Opus 5.5 alone at default effort for about the same score.

On the Claude Platform, multiagent orchestration in Claude Managed Agents provides a coordinator plus a roster of workers. Skip it for a single chain of dependent steps or work that fits in one context window, since planning and merging cost more than they save.

## Example split in one project

Fable 5.2: JavaScript → TypeScript migration. Opus 5.5: new subscription feature (starts after Fable's code lands). Sonnet 5.5: product pages. Haiku 5.5: customer-message triage.

## Routing

When the rule is simple (one model per step type), let code choose the model rather than asking a model to.
