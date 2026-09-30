---
name: model-selection
description: Use before delegating work to a subagent, or when a task is large, repetitive or exploratory, to pick the cheapest model that can still do it well (haiku, sonnet, opus, fable). Use when the user says "Token sparen", "welches Modell", "model routing", or "is this worth a bigger model". Not for choosing a model inside an application you are building (use engineering/llm-cost-optimizer).
---

# Model Selection

Goal: spend the expensive model's tokens only where its judgement changes the outcome.

## Limits (be honest about them)

- The **main session's model cannot be switched by a skill**. The user does that with `/model`. If the whole session is mis-sized, say so once and suggest the switch.
- What Claude controls is the `model` parameter of the **Agent tool** (`haiku` | `sonnet` | `opus` | `fable`). Routing means: keep judgement in the main session, push volume work to cheaper subagents.
- Cheaper models cost more when they fail and force a redo. When unsure, go one tier up, not down.

## Decision table

| Task shape | Model |
|---|---|
| Grep/glob sweeps, file listing, log skimming, counting, format conversion, boilerplate, running tests and summarizing output | `haiku` |
| Normal implementation, bug fixes with a clear repro, refactors, code search plus explanation, doc writing, test writing | `sonnet` (default) |
| Architecture and design trade-offs, ambiguous requirements, hard debugging after sonnet failed once, security-sensitive review, plan critique | `opus` |
| Long autonomous runs, multi-system reasoning, work where a wrong answer is expensive to detect or undo | `fable` |

Verify current prices and model lineup (`claude-api` skill or the pricing page) before quoting numbers; do not answer cost questions from memory.

## Procedure

1. Classify the task by the table. Ask: *would a wrong answer be caught cheaply?* If yes, go cheaper. If no, go higher.
2. Split mixed work: scout with `haiku`, implement with `sonnet`, review the diff with `opus`. Give each subagent a self-contained prompt; subagents start cold.
3. Ask subagents for **short, structured reports** (paths, findings, verdict), not file dumps. The report is what lands in the main context.
4. Do trivial one-step work directly. A subagent costs a full context bootstrap; below roughly three tool calls, spawning loses.
5. **Escalate once**: if a cheaper model returns a wrong or empty result, rerun that step one tier up. Do not retry the same tier.
6. State the choice in one line when delegating ("Haiku scout: mechanical search"), so the user can veto.

## Anti-patterns

- Opus or Fable for grep, listings, or formatting.
- Haiku for security review, migrations, or anything without a cheap correctness check.
- Spawning many parallel subagents that each re-read the same large files.
- Downgrading a step that already failed once.

## Existing community options

- [AqueGen/model-routing](https://github.com/AqueGen/model-routing): pinned subagents plus a dispatch counter that measures what stayed off the session model.
- [soumabali/token-router](https://github.com/soumabali/token-router): automatic 3-tier routing via hooks.
Review any hook-based router before installing; it sits on every prompt.
