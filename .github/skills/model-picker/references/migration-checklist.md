# Compatibility checklist before swapping models

Every current model supports tool use and image input, but changing only the model name is not always safe. Test these first.

## Preserved thinking (Fable 5.2)

Fable 5.2 extends preserved thinking to every API account. The API returns a 400 if anything before a Fable 5.2 thinking block has changed (system prompt, tools, or an earlier message), unless you opt in to dropping the affected thinking.

- Claude Code, Claude Agent SDK and Claude Managed Agents handle this; no change needed.
- Custom code that edits history: keep it append-only and test before switching.

## Thinking

- Opus 5.5 and Fable 5.2 always think; a request that turns thinking off returns an error.
- Sonnet 5.5 thinks by default; `between_tools` limits thinking to gaps between tool calls.
- Haiku 5.5 thinks by default; thinking can be turned off at high effort or below.
- On all of them, read content blocks by type, because a response can begin with a thinking block.

## Forced tool use

`tool_choice` of `any` or `tool` returns 400 on Sonnet 5.5, Opus 5.5 and Fable 5.2. Send `auto`, mark the tool `strict: true`, and say in the prompt when to use it.

## Computer use

Opus 5.5, Sonnet 5.5, Haiku 5.5 and Fable 5.2 support computer use only through the `computer_toolset_20260801` toolset on the Claude API and Claude Models on Google Cloud (Google managed).

## Haiku 4.5 → Haiku 5.5

- Replace `budget_tokens` with an effort level.
- Drop non-default `temperature`, `top_p`, `top_k` (they return 400).
- End messages with a user turn; a final assistant turn for the model to continue returns 400.

## Safety fallbacks

Opus 5.5, Sonnet 5.5 and Fable 5.2 run safety classifiers that most often flag cybersecurity and biology content. In Claude Code a flagged request can re-run on an older model, and a transcript notice says which. Fable 5.2 keeps Fable 5.1's fallbacks: flagged cybersecurity requests fall back to Opus 4.8, flagged biology requests to Opus 5.

## Build for the strictest model

If one system runs several models, write every request for the strictest of them so a step can move up or down a tier without a code change:

- history append-only
- `tool_choice: auto` with strict tools
- content blocks read by type
- prompts and tool descriptions precise enough for the most literal model
