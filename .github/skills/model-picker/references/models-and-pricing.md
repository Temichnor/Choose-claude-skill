# Models and pricing (snapshot from the Nov 2026 playbook)

Verify against current Claude docs before quoting in a final recommendation. Prices are Claude API list prices.

## Side by side

| | Haiku 5.5 | Sonnet 5.5 | Opus 5.5 | Fable 5.2 |
|---|---|---|---|---|
| Built for | High-volume, cost-sensitive tasks | Fast, capable help with well-scoped tasks | Daily driver for complex coding and knowledge work | Most ambitious, open-ended technical work |
| Relative latency | Fastest | Fast | Moderate | Moderate |
| Input / output per M tokens | $0.10 / $0.50 up to 100K-token prompts; $0.50 / $2.50 over 100K | $2 / $10 | $4 / $20 | $8 / $40 |
| Cache reads per M tokens | $0.01 up to 100K; $0.05 over 100K | $0.10 | $0.20 | $0.20 |
| Context window | 1M | 1M | 1M | 1M |
| Max output | 128K | 128K | 128K | 128K |
| Thinking on the API | Adaptive, on by default, can be turned off | Adaptive, on by default; `between_tools` skips upfront thinking | Adaptive, always on | Adaptive, always on |
| Default effort on API | medium | high (medium in Claude Code) | medium | high (medium in Claude Code) |
| Fast mode | No | No | Yes | Yes |

`/effort status` in Claude Code shows the effective default.

Claude Mythos 5.2: most capable for cybersecurity and life-sciences research, only through verification programs.

## Useful ratios

- Opus 5.5 = half of Fable 5.2 per input/output token; Sonnet 5.5 = half of Opus 5.5.
- Haiku 5.5 short-prompt input is 1/20 of Sonnet 5.5; long-prompt tier is still 1/4 of Sonnet 5.5 per token.
- Cache read cost per turn (Opus 5.5): about $0.03 at 150K context, about $0.004 at 20K.
- Speed: Fable 5.2 output is up to 30% faster than Fable 5.1; Sonnet 5.5 is 30% faster than Sonnet 5.

## Benchmark data points cited

- Terminal-Bench 3.0: prior Fable 5.1 at max 58.0% vs Opus 5.5 at high 58.9% with half the tokens.
- FrontierCode 2.0: Fable 5.2 at high 45.3% vs Opus 5.5 at max 45.5%, Fable ~half the cost per task.
- SWE-bench Pro, Opus 5.5 vs its own high setting: medium about -2.5 pts at ~70% cost; low about -8 pts at ~⅓ cost; xhigh about +1.4 pts at 2.5× cost.
- SWE-bench Pro subset: Opus 5.5 low with failures re-run at high ≈ 97% at ~$0.17/task vs 95.3% at $0.29 with all at high.
- Internal defect search over ~22M tokens of source: Opus 5.5 lead + up to 25 parallel Sonnet 5.5 workers cost ~⅓ less than Opus 5.5 alone at default effort, similar score.
- Internal coding benchmark: Opus 5.5 high + Fable 5.1 advisor scored 90.1% at $2.92/attempt, +1.7 pts over Opus 5.5 alone for ~2.1× cost, within run-to-run noise.
